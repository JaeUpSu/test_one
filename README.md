HTML 기능별로 대충 구성하기

chakra-ui
react-query
mutation
react-hook-form

## chakra ui

    Chackra UI Install
    Style props
    Components in Chakra UI
    Theming
    Responsiveness

    ---

    npm i @chakra-ui/react @emotion/react @emotion/styled framer-motion

    ---

    [index.jsx]

    ...
    import React from 'react'

    // [add] import
    import { ChakraProvider } from '@chakra-ui/react'

    ReactDOM.render(
        <React.StrictMode>
            <ChakraProvider>
                <App />
            </ChakraProvider>
        </React.StrictMode>
        document.getElementById('root')
    );

    ---

    필요한 컴포넌트, style props 등 적용하여 사용 (BootStrap 같음)
    hover 나 focus 등 다 있음

## react-query

    - 캐싱 하기
    - 동기화 하기
    - 업데이트 하기
    - Server state 를 쉽게 가져오기

    - React App 구성
        비동기 데이터를 저장하고 제공하는 것 의미
        (기존 라이브러리는 비동기 또는 서버 상태 작업 그다지 적합)

        - 캐싱
        - 동일한 데이터를 여러요청 -> 단일요청 (중복 제거)
        - 백그라운드에서 오래된 데이터 업데이트
        - 데이터가 오래된 시기 파악
        - 가능한 한 빨리 데이터 업데이트 반영
        - 페이지 매김 및 지연 로드 데이터와 같은 성능 초기화
        - 서버 상태의 메모리 및 가비지 수집 관리
        - 구조적 공유를 통한 쿼리 결과 메모

    - 서버 상태 관리를 위한 최고의 라이브러리
    - 구성이 필요 없이 즉시 사용 가능

    ----

    npm i react-query

    ----

    import {
            QueryClient,
            QueryClientProvider,
            useQuery,
            useMutation,
            useQueryClient
            } from 'react-query'

    import { getTodos, postTodo } from '../my-api'


    const queryClient = new QueryClient();

    export default function App() {
        return (
            <QueryClientProvider client={queryClient}>
                <Example />
            </QueryClientProvider>
        )
    }

    function Example() {
        const {isLoading, error, data} = useQuery('repoData', () =>
            fetch('https://api.github.com/repos/tannerlinsley/react-query').then(res => res.json())
        )

        if (isLoading) return 'Loading...'

        if (error) return 'An error has occurred: ' + error.message

        return (
            <div>
                <h1>{data.name}</h1>
                <p>{data.description}</p>
                <strong>{data.subscribers_count}</strong>{' '}
                <strong>{data.stargazers_count}</strong>{' '}
                <strong>{data.forks_count}</strong>{' '}
            </div>
        )
    }

    function Todos() {
        const queryClient = useQueryClient()
        const query = useQuery('todos',getTodos)

        // Server State를 변경시키는 hook (create, update, delete)
        const mutation = useMutation(postTodo, {

            // mutationFn 성공했을 때 실행
            onSuccess: () => {
                // 해당 쿼리를 invalidate 하고 refetch 함
                queryClient.invalidateQueries('todos')
            },
        })

        return (
            <div>
                <ul>
                    {query.data.map(todo => (
                        <li key={todo.id}>{todo.title}</li>
                    ))}
                </ul>
                <button
                    onClick={()=>{

                        // mutationFn 을 trigger 하는 함수
                        mutation.mutate({
                            id: Date.now(),
                            title: 'Do Laundary',
                        })
                    }}
                >
                    Add Todo
                </button>
            </div>
        )
    }

    render(<App />, document.getElementById('root'))

## tanstack/react-query

    쿼리는 서버에서 데이터를 가져오기 위해
    프로미스 기반의 메서드(GET, POST...) 와 사용
    (데이터를 서버에서 수정하기 위해서는 Mutations을 사용한다.)

## react-hook-form

    - 양식을 쉽게 관리할 수 있는 사용자 지정 후크
    - 하나의 객체를 선택적 인수로 사용

    - useForm() 훅(hook) 함수를 부르기
    - 결과 객체로 부터 register(), handleSubmit() 함수를 얻기
    - register() 함수를 이용하여 각 입력란을 등록
    - handleSubmit() 함수를 이용하여 form 요소에서 발생하는 submit 이벤트를 처리

    - useForm() 훅(hook) 함수가 반환하는 객체의 formState 속성
      양식이 현재 어떤 상태인지를 담는데,
      formState으로 부터 isSubmitting 속성을 읽어서 양식이
      현재 제출 중인 상태인지 아닌지 알기

    ---

    // 동기일 때
    useForm({
        defaultValues: {
            firstName: '',
            lastName: ''
        }
    })

    // 비동기일 때
    useForm({
        defaultValues: async () => fetch('/api-endpoint');
    })

    ---

    - useForm 을 이용한 LoginForm 만들기

    import { useForm } from "react-hook-form";

    function LoginForm({
      onSubmit = async (data) => {
        await new Promise((r) => setTimeout(r, 1000));
        alert(JSON.stringify(data));
      }
    }) {
      const {
        register,
        handleSubmit,
        formState: { isSubmitting, isDirty, errors }
      } = useForm();

      return (
        <form onSubmit={handleSubmit(onSubmit)}>
          <label htmlFor="email">이메일</label>
          <input
            id="email"
            type="text"
            placeholder="test@email.com"
            aria-invalid={!isDirty ? undefined : errors.email ? "true" : "false"}
            {...register("email", {
              required: "이메일은 필수 입력입니다.",
              pattern: {
                value: /\S+@\S+\.\S+/,
                message: "이메일 형식에 맞지 않습니다."
              }
            })}
          />
          {errors.email && <small role="alert">{errors.email.message}</small>}
          <label htmlFor="password">비밀번호</label>
          <input
            id="password"
            type="password"
            placeholder="****************"
            aria-invalid={!isDirty ? undefined : errors.password ? "true" : "false"}
            {...register("password", {
              required: "비밀번호는 필수 입력입니다.",
              minLength: {
                value: 8,
                message: "8자리 이상 비밀번호를 사용하세요."
              }
            })}
          />
          {errors.password && <small role="alert">{errors.password.message}</small>}
          <button type="submit" disabled={isSubmitting}>
            로그인
          </button>
        </form>
      );
    }

    export default LoginForm;

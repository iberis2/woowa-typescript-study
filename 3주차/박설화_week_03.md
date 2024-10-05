
# 타입 검증, api 데이터 모킹, 에러 핸들링

우아한 타입스크립트 6, 7장에서 줍줍한 사용해보면 좋을 것 같은 라이브러리 & 코드 정리

## 1. 런타임에서 응답 타입 검증하기
> ## [superstruct](https://www.npmjs.com/package/superstruct)
> 공식 문서 : https://docs.superstructjs.org/\
> npm : https://www.npmjs.com/package/superstruct\
> github : https://github.com/ianstormtaylor/superstruct

- 정적 타입 검증과 런타임 데이터 검증을 동시에 제공한다.
- 사용자가 검증하고 싶은 유효성 검사를 쉽게 커스텀하여 정의할 수 있는 유연성을 가지고 있다.
  - zod, yup 와 마찬가지로 react-hook-form 에서 resolver 로 사용할 수도 있다.
  - zod 보다 번들 사이즈가 작다. (1/3 배)
- 단점은 비동기 지원이 안된다는 점

```ts
import { struct } from 'superstruct';

// API 응답 user 의 타입 정의
const User = struct({
  id: 'number',
  name: 'string',
  age: 'number',
});

async function fetchUser(){
  const { data } = await axios.get('/api/user');
  
  try {
    User(data);  // 데이터 검증  
    console.log('Valid data', data); // { id: 1, name: 'Alice', age: 25 };
    return data;
  } catch (error) { // 데이터 타입 다른 경우 { id: 1, name: null, age: 25 };
    console.log('Invalid data:', error);
  }
}
```



비동기 작업 필요없이 간단한 유효성 검증 혹은 api response 로 온 데이터 타입 검증이 필요할 때 한 번 적용해보면 좋을 것 같다.

-----
참고 : [[Typescript] Zod에서 Superstruct로의 전환기](https://blog.betaman.kr/131)

## 2. API 데이터 모킹

### NextApiHandler
책의 NextApiHandler 설명은 pages router 버전의 예제인 것 같아서 
Next js 14버전의 app router 기준의 예제를 만들어 봤다.
[🔗 Next js 공식문서 router handler](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)

<iframe src="https://codesandbox.io/p/devbox/2y74r8?embed=1&file=%2Fapp%2Fpage.tsx"
     style="width:100%; height: 500px; border:0; border-radius: 4px; overflow:hidden;"
     title="next api handler"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

> ## [MSW](https://www.npmjs.com/package/msw)
> npm : https://www.npmjs.com/package/msw
> 깃헙 : https://github.com/mswjs/msw
> 공식문서 : https://mswjs.io/
 -----
> 참고
> [올리브영 테크 블로그 : Next.js에서 MSW(Mock Service Worker)로 네트워크 Mocking하기](https://oliveyoung.tech/blog/2024-01-23/msw-frontend/)

원래 빠르고 가볍게 mock data 를 만들어서 사용할 때 [json-server](https://www.npmjs.com/package/json-server) 를 주로 사용했었는데, 책에서도, 스터디 팀원들도 msw 를 많이 추천하길래 관심이 생겼다.
다음 프로젝트에서 한 번 사용해보면 좋을 것 같다.
json-server 보다 용량은 4배정도 크지만,
네트워크 탭으로도 실제 요청이 오고 가는 걸 확인할 수 있고, 실제 서버와 작업이 이루어지는 것과 거의 유사한 환경을 만들 수 있다고 한다.

![](https://velog.velcdn.com/images/iberis/post/329000cb-275f-4ddf-8ae6-df1c3207fbff/image.png)

> ## [axios-mock-adapter](https://www.npmjs.com/package/axios-mock-adapter)
> 깃헙 & 공식문서 : https://github.com/ctimmerm/axios-mock-adapter
> npm : https://www.npmjs.com/package/axios-mock-adapter
- Axios 요청을 가로채서 요청에 대한 응답 값을 대신 반환한다.
    - MockAdapter 객체를 생성하고, 해당 객체를 사용하여 모킹할 수 있다.
- mock API의 주소가 따로 필요하지 않다

## 3. 에러 서브클래싱하기
여러 에러 상황이 있을 때 에러 인스턴스가 무엇인지에 따라 에러 처리 방식을 다르게 구현할 수 있다.

1. OrderHttpError, NetworkError, UnauthorizedError 각 에러 타입 별 클래스를 정의한다.
```ts
class OrderHttpError extends Error {
  constructor(message?: string, response?: AxiosResponse<ErrorResponse>) {
    super(message);
    this.name = "OrderHttpError";
  }
}

class NetworkError extends Error {
  constructor(message = "") {
    super(message);
    this.name = "NetworkError";
  }
}

class UnauthorizedError extends Error {
  constructor(message: string, response?: AxiosResponse<ErrorResponse>) {
    super(message, response);
    this.name = "UnauthorizedError";
  }
}
```

2. 각 에러 class 를 전달하여 axios error handler 를 만들고 axios 의 인터셉터에 전달한다.
```ts
const httpErrorHandler = (
  error: AxiosError<ErrorResponse> | Error
): Promise<Error> => {
  let promiseError: Promise<Error>;

  if (axios.isAxiosError(error)) {
    if (Object.is(error.code, "ECONNABORTED")) { // 서버와 연결 종료
      promiseError = Promise.reject(new TimeoutError());
    } else if (Object.is(error.message, "Network Error")) { // 네트워크 에러
      promiseError = Promise.reject(new NetworkError());
    } else {
      const { response } = error as AxiosError<ErrorResponse>;
      switch (response?.status) {
        case HttpStatusCode.UNAUTHORIZED: // 권한 없음 에러
          promiseError = Promise.reject(
            new UnauthorizedError(response?.data.message, response) 
          );
          break;
        default:
          promiseError = Promise.reject(
            new OrderHttpError(response?.data.message, response) // 주문 실패 에러
          );
      }
    }
  } else {
    promiseError = Promise.reject(error);
  }

  return promiseError;
};


function createAuthAxiosInstance() {
  const instance = axios.create(instanceOptions)
  instance.interceptors.response.use(
   response:AxiosResponse) => response,
   httpErrorHandler
  )
  
  return instance;
}
export const axiosInstance = createAuthAxiosInstance();
```

3. 에러를 잡아서 각 class 별로 다르게 에러를 처리할 함수를 정의한다
```ts

const onActionError = (
  error: unknown,
  params?: Omit<AlertPopup, "type" | "message">
) => {
  if (error instanceof UnauthorizedError) {
    onUnauthorizedError(
      error.message,
      errorCallback?.onUnauthorizedErrorCallback
    );
  } else if (error instanceof NetworkError) {
    alert("네트워크 연결이 원활하지 않습니다. 잠시 후 다시 시도해주세요.", {
      onClose: errorCallback?.onNetworkErrorCallback,
    });
  } else if (error instanceof OrderHttpError) {
    alert(error.message, params);
  } else if (error instanceof Error) {
    alert(error.message, params);
  } else {
    alert(defaultHttpErrorMessage, params);
  }
```

4. api 요청의 catch 문에 에러 처리함수를 적용한다.
```ts
  const getOrderHistory = async (page: number): Promise<History> => {
    try {
      const { data } = await axiosInstance.get(`https://some.site?page=${page}`);
      const history = await JSON.parse(data);

      return history;
    } catch (error) {
      onActionError(error); // 에러 처리 함수 적용
    }
  };
};
```

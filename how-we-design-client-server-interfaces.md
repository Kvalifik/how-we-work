# How We Design Client-Server Interfaces
Having a coherent code standard between clients and servers makes it much easier to switch between projects.
All of our backends communicate through REST-interfaces, and we strive to keep our interface compliant with the REST standard.

## Result types
We always return JSON objects from requests. We make sure that all of our requests always complies with the `APIResult<T>` type defined as such:
~~~typescript
type APISuccess<T> = { ok: T }
type APIError = {
    statusCode: number,
    message: string
}
type APIResult<T> = APISuccess<T> | APIError
~~~
In this case, any exceptions thrown by NestJS will be compliant with the `APIError` type. The result type forces the API consumer to consider the case where `ok` is undefined e.g. when an error has been thrown.

## Request Libraries
We always use `axios` as our HTTP client. We always use `react-query` as a request client to handle calling `axios`, caching results, retrying requests, etc.
We divide `react-query` calls into hooks named after their respective entities in the `hooks` folder. See [Turningtables Frontend](https://github.com/Kvalifik/turningtables-frontend) for a great example.

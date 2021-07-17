---
title: Rate limiting request
description: Tutorial on how to limit rate of request with  utilities provided by Fano Framework
---

<h1 class="major">Rate limiting request</h1>

## Why limit request?

![Rate limiting](/assets/images/rate-limit.jpg){:class="image fit"}

(Image by Ludovic Charlet [unsplash.com](https://unsplash.com/photos/CGWK6k2RduY)).

While developing API, we want to be able to maximize our application so that it handles request as many as it can. But we also want to avoid client overusing API and cause denial of service to other client or we want to allow paid clients to have bigger number of requests quota than free clients.

You will define a maximum number of request in a given amount of time. When clients reached maximum value, your application answers HTTP error code `429 Too Many Requests`.

## TLimitStatus record

`TLimitStatus is internal data structure that is used to maintain rate-limiting data and declared as shown below

```
type

     TLimitStatus = record
        limitReached : boolean;
        limit : integer;
        remainingAttempts : integer;

        //timestamp when counter will be reset
        resetTimestamp : integer;

        //number of seconds after counter will be reset
        retryAfter : integer;
    end;
```

- `limitReached` is boolean value to identify if current request exceeds its limit.
- `limit` is maximum number of operation allowed.
- `remainingAttempts` is number of operation available. If it equals 0 then limit is reached.
- `resetTimestamp` is timestamp when remaining attempts will be reset back to maximum operation allowed.
- `retryAfter` is number of seconds before remaining attempts in reset.

## Rate-limit middleware

Fano Framework provides `TThrottleMiddleware` and `TNonBlockingThrottleMiddleware` to limit number of request to certain application routes or global to all routes. The latter will not block request, it only tests if limit is reached and modify request to include rate limit status. Next middleware or controller can decides what to do with it. This is useful for example, you want to count excess number of request as additional billing charge instead of blocking request.

To use this [middleware](/middlewares), register its factory class [`TThrottleMiddlewareFactory`](https://github.com/fanoframework/fano/blob/master/src/Libs/Throttle/Implementations/Factories/ThrottleMiddlewareFactoryImpl.pas) or [`TNonBlockingThrottleMiddlewareFactory`](https://github.com/fanoframework/fano/blob/master/src/Libs/Throttle/Implementations/Factories/ThrottleMiddlewareFactoryImpl.pas) to [service container](/dependency-container) as shown in following code.

```
container.add(
    'throttle-one-request-per-sec',
    TThrottleMiddlewareFactory.create()
);
```
and then attach middleware to one or more [routes](/working-with-router).

```
router.get(
    '/',
    container['homeController'] as IRequestHandler
).add(container['throttle-one-request-per-sec'] as IMiddleware);
```
### Non blocking middleware
For non blocking throttle middleware
```
container.add(
    'throttle-one-request-per-sec',
    TNonBlockingThrottleMiddlewareFactory.create()
);
```
After attaching `TNonBlockingMiddleware` to a route, everytime route is executed then
request contains additional rate-limiting data (`TLimitStatus`) which can be retrieved from request object in next middleware or controller using `getParam()` method as shown below.

```
function TMyController.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
var
    limitReached : boolean;
begin
    if (tryStrtoBool(request.getParam('__limitreached'), limitReached)) and
        limitReached then
    begin
        response.body().write(
            'limit reach retry after:' + request.getParam('__retry_after')
        );
    end else
    begin
        response.body().write('Home controller');
    end;
    result := response;
end;
```

By default, this middleware add following key parameters

- `__limit_reached`, this key is related to field `limitReached` of `TLimitStatus`.
- `__limit`, this key is related to field `limit` of `TLimitStatus`.
- `__remaining_attempts`, this key is related to field `remainingAttempts` of `TLimitStatus`.
- `__reset_timestamp`, this key is related to field `resetTimestamp` of `TLimitStatus`.
- `__retry_after`, this key is related to field `retry_after` of `TLimitStatus`.

Using getParam(), data is always in string so you need to convert it to its correct type.
Other means to query rate limiting data is by testing if request object is `IThrottleRequest` instance. [This interface](https://github.com/fanoframework/fano/blob/master/src/Libs/Throttle/Contracts/ThrottleRequestIntf.pas) exposes additional property `limitStatus`.

```
function TMyController.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
var
    throttleRequest : IThrottleRequest;
begin
    if request is IThrottleRequest then
    begin
        throttleRequest := request as IThrottleRequest;
        if throttleRequest.limitStatus.limitReached then
        begin
            response.body().write(
                'limit reach retry after:' +
                    intToStr(throttleRequest.limitStatus.retryAfter)

            );
        end else
        begin
            response.body().write('Home controller');
        end;
    end else
    begin
        response.body().write('Home controller');
    end;
    result := response;
end;
```

## Change number of requests

By default when you use `TThrottleMiddlewareFactory` as shown above, throttle middleware allows 1 request per second only. If you make another request before 1 second elapse, you get HTTP 429 error.

To change number of requests calls following factory methods:

- `ratePerSecond()`, set number of requests per second.
- `ratePerMinute()`, set number of requests per minute.
- `ratePerHour()`, set number of requests per hour.
- `ratePerDay()`, set number of requests per day.
- `rate()`, set number of requests of given interval in seconds.

Except `rate()` method, which requires two parameters, other methods require one parameter, i.e integer value of maximum number of requests. All `rate*()` methods return current factory instance so you can chain its call.

For example to set maximum 2 requests per second
```
container.add(
    'throttle-two-request-per-sec',
    TThrottleMiddlewareFactory.create()
        .ratePerSecond(2)
);
```

To set custom interval of 10 requests per 30 minutes
```
const
    NUM_SECONDS_IN_30_MINUTES = 30 * 60;

container.add(
    'throttle-ten-request-per-30-min',
    TThrottleMiddlewareFactory.create()
        .rate(10, NUM_SECONDS_IN_30_MINUTES)
);
```

`rate*()` methods as basically just set internal field of type `TRate`, which is, record declared as follows,

```
TRate = record
    //number of operations allowed
    operations : integer;

    //interval in seconds
    interval : integer;
end;
```

## Change how requests are identified
To be able to tell which clients exceed limit, throttle middleware need to be able to indentify requests using instance of `IRequestIdentifier` interface.

Currently, Fano Framework provides two implementations of this interface.

- `TIpAddrRequestIdentifier` which identifies request based on IP address.
- `TSessionRequestIdentifier` which identifies request based on [session ID](/working-with-session).

By default, request is identified using its IP address. To change request identifier instance, call `requestIdentifier()` method of factory and pass new instance

```
container.add(
    'throttle-one-request-per-sec',
    TThrottleMiddlewareFactory.create()
        .requestIdentifier(TSessionRequestIdentifier.create())
);
```
`requestIdentifier()` returns current factory instance so that you can chain with other methods,

```
container.add(
    'throttle-one-request-per-sec',
    TThrottleMiddlewareFactory.create()
        .requestIdentifier(TSessionRequestIdentifier.create())
        .ratePerSecond(1)
);
```

If you need to use different ways to identify request, for example using unique key passed as query string or POST parameter, you can create a class which implements `IRequestIdentifier` interface and implement its `getId()` method. For example

```
unit MyRequestIdentifierImpl;

interface

{$MODE OBJFPC}
{$H+}

uses

    RequestIntf,
    RequestIdentifierIntf;

type

    TMyRequestIdentifier = class (TInterfacedObject, IRequestIdentifier)
    public
        (*!------------------------------------------------
         * get identifier from request
         *-----------------------------------------------
         * @param request request object
         * @return identifier string
         *-----------------------------------------------*)
        function getId(const request : IRequest) : shortstring; override;
    end;

implementation

(*!------------------------------------------------
 * get identifier from request
 *-----------------------------------------------
 * @param request request object
 * @return identifier string
 *-----------------------------------------------*)
function TMyRequestIdentifier.getId(
    const request : IRequest
) : shortstring;
begin
    result := request.getParam('accesskey');
end;
```
You can also create class inherit from `TAbstractRequestIdentifier` and implement its abstract method `getId()`.

## Change rate limiter

Throttle middleware depends on instance of `IRateLimiter` interface to do actual test of request limitation. Currently Fano Framework provides

- `TMemoryRateLimiter` which tracks requests on memory. This implementation can not be used in CGI application as CGI application is created for each request.
- `TDbRateLimiter` which tracks request in relational database table.
- `TDecoratorRateLimiter` which decorates other `IRateLimiter` instance.

Development of other type rate limiter such as rate limiter which keeps track request in Redis or MongoDB is planned.

### Memory storage rate limiter
By default, `TThrottleMiddlewareFactory` uses `TMemoryRateLimiter`. Internal implementation of this class store data using hash map in memory.

### Database storage rate limiter
You need `TDbRateLimiter` to [use relational database](/database) such as MySQL, PostgreSQL, Firebird or SQLite to track requests.

`TDbRateLimiter` class requires instance if `IRdbms` interface which responsible to do actual database operation. You need to tell what table to use and also column name of identifier column, operation column and reset timestamp.

```
container.add(
    'throttle-one-request-per-minute',
    TThrottleMiddlewareFactory.create()
        .ratePerMinute(1)
        .rateLimiter(
            TDbRateLimiter.create(
                container['db'] as IRdbms,
                'rates',
                'id',
                'operation',
                'reset_timestamp'
            )
        )
);
```
Identifier column stores identifier related to request, for example IP address, session ID or other unique value and it is expected to be VARCHAR column and primary key.

Operation column stores integer value number of operation . Reset timestamp column stores integer value of timestamp. Following SQL is minimal schema for table

```
CREATE TABLE rates
(
    id VARCHAR(100) PRIMARY KEY NOT NULL,
    operation INT(11) NOT NULL,
    reset_timestamp INT(11) NOT NULL
);
```

### Decorator rate limiter
If you need to modify rate dynamically, for example, each client type has its own maximum rate, you can create rate limiter inherit from `TDecoratorRateLimiter` and modify its `limit()` method as shown in following example.

```
unit MyRateLimiterImpl;

interface

{$MODE OBJFPC}
{$H+}

uses

    RateLimiterIntf,
    RateTypes,
    DecoratorRateLimiter;

type

    TMyRateLimiter = class (TDecoratorRateLimiter)
    public
        (*!------------------------------------------------
         * check if number of operations identified by identifier
         * not exceed rate configuration
         *-----------------------------------------------
         * @param identifier unique identifier
         * @param rate rate configuration
         * @return limit status
         *-----------------------------------------------*)
        function limit(
            const identifier : shortstring;
            const rate : TRate
        ) : TLimitStatus; override;

    end;

implementation

    (*!------------------------------------------------
     * check if number of operations identified by identifier
     * not exceed rate configuration
     *-----------------------------------------------
     * @param identifier unique identifier
     * @param rate rate configuration
     * @return limit status
    *-----------------------------------------------*)
    function TMyRateLimiter.limit(
        const identifier : shortstring;
        const rate : TRate
    ) : TLimitStatus;
    var customRatePerUser : TRate;
    begina
        customRatePerUser := rate;

        //TODO: load maximum number of request from
        //database with identifier as primary key
        //customRatePerUser := getRateFromDatabase(identifier);

        result := inherited limit(identifier, customRatePerUser);
    end;

end.
```

You can register throttle middleware with memory rate limiter but rate is loaded dynamically from [database](/database) as shown below.

```
container.add(
    'throttle-one-request-per-sec',
    TThrottleMiddlewareFactory.create()
        .rateLimiter(
            TMyRateLimiter.create(
                TMemoryRateLimiter.create()
            )
        )
);
```
## Rate-limiting video tutorial


[![Rate limiting video tutorial](/assets/images/video-preview.png){:class="image fit"}](https://youtu.be/dfmArIN4s-o "Rate limiting video tutorial")

Rate-limiting video tutorial explains how to use rate limit middleware to restrict number of request client can make.

## Explore more

- [Utilities](/utilities)
- [Middlewares](/middlewares)
- [TThrottleMiddleware source](https://github.com/fanoframework/fano/blob/master/src/Libs/Throttle/Implementations/ThrottleMiddlewareImpl.pas)
- [TThrottleMiddlewareFactory source](https://github.com/fanoframework/fano/blob/master/src/Libs/Throttle/Implementations/Factories/ThrottleMiddlewareFactoryImpl.pas)
- [Rate-limiting demo application](https://github.com/fanoframework/fano-rate-limiting)
- [Rate-limiting demo application with MySQL as storage](https://github.com/fanoframework/fano-rate-limiting-db).

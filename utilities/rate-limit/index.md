---
title: Rate limiting request
description: Tutorial on how to limit rate of request with  utilities provided by Fano Framework
---

<h1 class="major">Rate limiting request</h1>

## Why limit request?

![Rate limiting](/assets/images/rate-limit.jpg)
(Image by Ludovic Charlet [unsplash.com](https://unsplash.com/photos/CGWK6k2RduY)).


While developing API, we want to be able to maximize our application so that it handles request as many as it can. But we also want to avoid client overusing API and cause denial of service to other client or we want to allow paid clients to have bigger number of requests quota than free clients.

You will define a maximum number of request in a given amount of time. When clients reached maximum value, your application answers HTTP error code `429 Too Many Requests`.

## Rate-limit middleware

Fano Framework provides `TThrottleMiddleware` to limit number of request to certain application routes or global to all routes. To use this [middleware](/middlewares), register its factory class (`TThrottleMiddlewareFactory`) to [service container](/dependency-container) as shown in following code.

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

## Change number of requests

By default when you use `TThrottleMiddlewareFactory` as shown above, throttle middleware allows 1 request per second only. If you create another request before 1 second elapse, you get HTTP 429 error.

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

## Change how requests are identified
To be able to tell which clients exceed limit, throttle middleware need to be able to indentify requests using instance of `IRequestIdentifier` interface.

Currently, Fano Framework provides two implementations of this interface.

- `TIpAddrRequestIdentifier` which identifies request based on IP address.
- `TSessionRequestIdentifier` which identifies request based on session ID.

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

If you need to use different ways to identify request, for example using unique key passed as query string or POST parameter, ypu can create a class which implements `IRequestIdentifier` interface and implement its `getId()` method. For example

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
- `TDecoratorRateLimiter` which decorates other `IRateLimiter` instance.

Development of other type rate limiter such as rate limiter which keeps track request in Redis or MySQL is planned.

By default, `TMemoryRateLimiter` is used. If you need to modify rate dynamically, for example, each client type has its own maximum rate, you can create rate limiter inherit from `TDecoratorRateLimiter` and modify its `limit()` method as shown in following example.

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
`TRate` is record declared as follows,

```
TRate = record
    //number of operations allowed
    operations : integer;

    //interval in seconds
    interval : integer;
end;
```

You can register throttle middleware with memory rate limiter but rate is load dynamically from database

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

## Explore more

- [Utilities](/utilities)
- [Middlewares](/middlewares)

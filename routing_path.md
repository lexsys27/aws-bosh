# Context path routing

In this excercise we will study context path routing feature of Cloud Foundry.

## Push apps

We will push two instances of `dora` application.

```
$ git clone https://github.com/cloudfoundry/cf-acceptance-tests.git
$ cd cf-acceptance-tests/assets/dora
$ cf push dora01 --no-route
$ cf push dora02 --no-route
```

## Create domain

```
cf create-domain <YOUR_ORG> test.cf.example.com
```

## Create and map routes

```
cf map-route dora01 cf.example.com --hostname test --path env/INSTANCE_GUID
cf map-route dora02 cf.example.com --hostname test --path id
```

Now if you go to [test.cf.example.com/id](test.cf.example.com/id) and to 
[test.cf.example.com/env/INSTANCE_GUID](test.cf.example.com/env/INSTANCE_GUID), 
you will see different instance ids. This is because requests are handled by 
different applications.

Note that this is not path rewrite like NGinx `proxy_path` does. Mapping `domain.com/path`
to an applications doesn't transform requests like `/path/A` to `/A`. So your app must be aware
of the whole `/path/A` construction to operate.

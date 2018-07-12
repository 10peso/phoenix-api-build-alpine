# Phoenix-api-build-alpine

This Dockerfile provides a base build image to use in multistage builds for Elixir and Phoenix apps.
It comes with the latest version of Alpine, Erlang and Elixir.
The image lacks any Node or NPM versions as it is expected to function
for applications that do not use asset pipeline.
It is intended for use in creating release images with or for your application.

No effort has been made to make this image suitable to run in unprivileged environments. It is expected
the user of this image will implement proper security practices themselves.

Many thanks to @beardedeagle, this repository was made using code and structure
of his [builder](https://github.com/beardedeagle/alpine-phoenix-builder).

## Usage

For your application:

```dockerfile
FROM 10peso/phoenix-api-build-alpine:1.0.0 as builder
ENV appdir /opt/your_app
WORKDIR ${appdir}
COPY . ${appdir}
RUN mix deps.get --only prod \
  && MIX_ENV=prod mix compile \
  && cd ${appdir} \
  && MIX_ENV=prod mix release --env=prod

FROM alpine:3.8
EXPOSE 4000
ENV appver 0.0.1
WORKDIR /opt/your_app
COPY --from=builder /opt/your_app/_build/prod/rel/your_app/releases/${appver}/your_app.tar.gz .
RUN apk add --no-cache bash libressl \
  && tar -xzvf your_app.tar.gz \
  && rm -rf your_app.tar.gz \
  && rm -rf /root/.cache \
  && rm -rf /var/cache/apk/*
CMD ["bin/your_app", "foreground"]
```

To boot straight to a iex prompt in the image:

```shell
$ docker run --rm -i -t 10peso/phoenix-api-build-alpine iex
```


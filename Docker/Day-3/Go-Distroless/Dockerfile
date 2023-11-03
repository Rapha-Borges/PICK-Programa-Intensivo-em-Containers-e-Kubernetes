FROM cgr.dev/chainguard/go:latest-dev AS build

WORKDIR /app

COPY . .

RUN go build -o main .

FROM cgr.dev/chainguard/static:latest

COPY --from=build /app/main /

CMD ["/main"]
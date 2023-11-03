FROM alpine:3.13 as base
COPY ./giropops-senhas /giropops-senhas
RUN apk add --no-cache py3-pip && pip install --no-cache-dir -r /giropops-senhas/requirements.txt

FROM base
COPY --from=base /giropops-senhas /giropops-senhas
WORKDIR /giropops-senhas
ENV REDIS_HOST=localhost
RUN rm -rf /root/.cache /root/.cargo /usr/local/include /usr/local/share
CMD flask run --host=0.0.0.0
FROM alpine:3.8 as watchdog

RUN apk --no-cache add curl \
    && echo "Pulling watchdog binary from Github." \
    && curl -sSL https://github.com/openfaas/faas/releases/download/0.9.6/fwatchdog > /usr/bin/fwatchdog \
    && chmod +x /usr/bin/fwatchdog \
    && cp /usr/bin/fwatchdog /home/app \
    && apk del curl --no-cache

FROM microsoft/dotnet:2.1-sdk as builder

# Supress collection of data.
ENV DOTNET_CLI_TELEMETRY_OPTOUT 1

WORKDIR /home/app/src/
COPY ./__MyInitialFunctionName__.csproj  .
RUN dotnet restore ./__MyInitialFunctionName__.csproj

COPY .  .

RUN dotnet publish -c release -o published

FROM microsoft/dotnet:2.1-runtime

COPY --from=watchdog /usr/bin/fwatchdog  /usr/bin/

# Create a non-root user
RUN addgroup --system app \
    && adduser --system --ingroup app app

WORKDIR /home/app/
COPY --from=builder /home/app/src/published .
RUN chown app:app -R /home/app

USER app

ENV fprocess="dotnet ./__MyInitialFunctionName__.dll"
EXPOSE 8080

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]

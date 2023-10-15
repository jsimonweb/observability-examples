FROM ${ARCH}mcr.microsoft.com/dotnet/aspnet:7.0. AS base
WORKDIR /app

FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:8.0-preview AS build
ARG TARGETPLATFORM

WORKDIR /src
COPY ["helloworld.csproj", "./"]
RUN dotnet restore "./helloworld.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "helloworld.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "helloworld.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
EXPOSE 3500
ENV ASPNETCORE_URLS=http://+:3500

ARG AGENT_VERSION=1.24.0

# Install dependencies
RUN apt-get update && apt-get install -y zip curl
RUN mkdir /otel
RUN curl -L -o /otel/otel-dotnet-install.sh https://github.com/open-telemetry/opentelemetry-dotnet-instrumentation/releases/download/v0.7.0/otel-dotnet-auto-install.sh
RUN chmod +x /otel/otel-dotnet-install.sh

ENV OTEL_EXPORTER_OTLP_ENDPOINT='ELASTIC_APM_SERVER_URL'
ENV OTEL_EXPORTER_OTLP_HEADERS='Authorization=Bearer ELASTIC_APM_SECRET_TOKEN'
ENV OTEL_LOG_LEVEL=info
ENV OTEL_METRICS_EXPORTER=otlp
ENV OTEL_RESOURCE_ATTRIBUTES=service.version=1.0,deployment.environment=production
ENV OTEL_SERVICE_NAME=helloworld
ENV OTEL_TRACES_EXPORTER=otlp

ENV OTEL_DOTNET_AUTO_HOME=/otel

RUN /bin/bash /otel/otel-dotnet-install.sh

# Provide necessary permissions for the script to execute
RUN chmod +x /otel/instrument.sh

# Run the env setup script and then start the dotnet application on container startup
ENTRYPOINT ["/bin/bash", "-c", "source /otel/instrument.sh && dotnet helloworld.dll"]

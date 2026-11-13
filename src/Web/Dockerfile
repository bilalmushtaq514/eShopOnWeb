# syntax=docker/dockerfile:1
# -----------------------------------------------------------------------------
# eShopOnWeb Web - multi-stage build (A-2)
#   build   : .NET 10 SDK, restore layer cached separately from source
#   runtime : ASP.NET 10 runtime only, runs as the image's non-root "app" user
# Build from the repo root:
#   docker build -f src/Web/Dockerfile --build-arg BUILD_VERSION=123-abc1234 -t eshop-web .
# -----------------------------------------------------------------------------
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src

# 1) Restore - only the files that influence package restore, so this layer is
#    reused until a project file or central package version changes.
COPY global.json Directory.Packages.props ./
COPY src/Web/Web.csproj                                                       src/Web/
COPY src/ApplicationCore/ApplicationCore.csproj                               src/ApplicationCore/
COPY src/Infrastructure/Infrastructure.csproj                                 src/Infrastructure/
COPY src/BlazorAdmin/BlazorAdmin.csproj                                       src/BlazorAdmin/
COPY src/BlazorShared/BlazorShared.csproj                                     src/BlazorShared/
COPY src/eShopWeb.AspireServiceDefaults/eShopWeb.AspireServiceDefaults.csproj src/eShopWeb.AspireServiceDefaults/
RUN dotnet restore src/Web/Web.csproj

# 2) Build + publish
COPY src/ src/
ARG BUILD_VERSION=local-dev
RUN dotnet publish src/Web/Web.csproj -c Release -o /app/publish --no-restore \
      -p:UseAppHost=false -p:InformationalVersion=${BUILD_VERSION}

# -----------------------------------------------------------------------------
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS runtime
ARG BUILD_VERSION=local-dev
LABEL org.opencontainers.image.source="https://github.com/bilalmushtaq514/eShopOnWeb" \
      org.opencontainers.image.version="${BUILD_VERSION}"
ENV APP_VERSION=${BUILD_VERSION} \
    ASPNETCORE_HTTP_PORTS=8080 \
    DOTNET_CLI_TELEMETRY_OPTOUT=1
WORKDIR /app
# Files stay owned by root (read-only for the app user): the process cannot
# modify its own binaries even if compromised.
COPY --from=build /app/publish .
# Non-root user (UID 1654) shipped with the official .NET 8+ images.
USER $APP_UID
EXPOSE 8080
ENTRYPOINT ["dotnet", "Web.dll"]

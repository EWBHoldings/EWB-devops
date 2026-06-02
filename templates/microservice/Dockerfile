FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

COPY ["src/MyService/MyService.csproj", "src/MyService/"]
RUN dotnet restore "src/MyService/MyService.csproj"

COPY . .
WORKDIR "/src/src/MyService"
RUN dotnet build "MyService.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "MyService.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .

# Run as a non-root user for security
RUN addgroup --system appgroup && adduser --system --ingroup appgroup appuser
USER appuser

ENTRYPOINT ["dotnet", "MyService.dll"]

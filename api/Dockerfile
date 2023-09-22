FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY ["./Basic3TierAPI/Basic3TierAPI.csproj", "./"]
RUN dotnet restore "Basic3TierAPI.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "./Basic3TierAPI/Basic3TierAPI.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "./Basic3TierAPI/Basic3TierAPI.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Basic3TierAPI.dll"]
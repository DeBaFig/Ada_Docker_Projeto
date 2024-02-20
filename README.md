![_]()  

![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)
![.NET](https://img.shields.io/badge/.NET-5C2D91?style=for-the-badge&logo=net&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
[![GitHub issues](https://img.shields.io/github/issues/DeBaFig/script_aprende_shell)](https://github.com/DeBaFig/script_aprende_shell/issues)
[![GitHub forks](https://img.shields.io/github/forks/DeBaFig/script_aprende_shell)](https://github.com/DeBaFig/script_aprende_shell/network)
[![GitHub stars](https://img.shields.io/github/stars/DeBaFig/script_aprende_shell)](https://github.com/DeBaFig/script_aprende_shell/stargazers)
[![GitHub license](https://img.shields.io/github/license/DeBaFig/script_aprende_shell)](https://github.com/DeBaFig/Ada_Docker_Projeto/blob/main/LICENSE)

# Projeto - Curso Conteinerização Ada Devª


- [Projeto - Curso Conteinerização Ada Devª](#projeto---curso-conteinerização-ada-devª)
  - [Sobre](#sobre)
  - [Objetivo](#objetivo)
  - [Como executar](#como-executar)
  - [Referências](#referências)
  - [Autora](#autora)
      

## Sobre

Projeto final do curso de Docker feito em .NET 8.0
Para criação das imagens foi usado como base o comando para criar um projeto em mvc em C#:

```bash
dotnet new mvc --auth Individual -o Ada_app
```

Depois eu adicionei um gitignore
```bash
cd Ada_app
dotnet new gitignore
```

Adicionei também um novo pacote de sql Postgres
```bash
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
```

Dentro do programa eu troquei o :
```cs
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection") ?? throw new InvalidOperationException("Connection string 'DefaultConnection' not found.");
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlite(connectionString));
builder.Services.AddDatabaseDeveloperPageExceptionFilter();
```
POR:
```cs
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection") ?? throw new InvalidOperationException("Connection string 'DefaultConnection' not found.");
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseNpgsql(connectionString));
builder.Services.AddDatabaseDeveloperPageExceptionFilter();
```
O "UseNpgsql" será usado para o Postgres 

A conexão string também tem que ser adicionada em appsettings.json:
Host: Deve ser o nome do Container escolhido no compose.yaml que tem o seu banco de dados

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=postgres_ada;Port=5432;Database=Nome_DB;UserName=postgres;Password=postgres"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

A parte de autenticação não vinha com o código pelo comando, então tive que anexar ao projeto utilizando os seguintes comandos:
```bash
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet aspnet-codegenerator identity -dc Ada_app.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout;Account.RegisterConfirmation"

```

Fiz algumas alterações utilizando um template gratuito da internet:

[Link para o template](https://startbootstrap.com/template/blog-post)

E como o peojeto vem estruturado para usar SQLite tem que substituir a propriedade nas migrações também:

onde ver:
```cs
b.Property<bool>

ex:
b.Property<bool>("PhoneNumberConfirmed").HasColumnType("INTEGER");
//SUBSTITUIR POR:
b.Property<bool>("PhoneNumberConfirmed").HasColumnType("BOOLEAN");
```

Criei o Dockerfile para minha imagem do .net:

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk as build
COPY . ./src
WORKDIR /src
RUN dotnet build -o /app
RUN dotnet publish -o /publish
FROM mcr.microsoft.com/dotnet/aspnet as base
COPY --from=build  /publish /app
WORKDIR /app
EXPOSE 80
CMD ["./Ada_app"]
```

Alterei algumas informações dentro do .csproj pois o .NET 8.0 necessitou:

```xml
<PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <EnvironmentName>Development</EnvironmentName>
    <Nullable>enable</Nullable>
    <ContainerImageName>ada_app_image</ContainerImageName>
    <ImplicitUsings>enable</ImplicitUsings>
    <UserSecretsId>aspnet-Ada_app-2181a886-5516-4008-a6b3-18ac39740617</UserSecretsId>
</PropertyGroup>
```

Criei o compose.yaml:

```yml
version: '3.9'

services:
  dotnet:
    container_name: 'Ada_app'
    image: 'ada_app_image'
    build:
      context: .
      dockerfile: ./Dockerfile
    restart: always
    ports:
     - "5000:80"
    environment: 
      ASPNETCORE_URLS : http://+
      ASPNETCORE_ENVIRONMENT : Development
    depends_on:
     - "postgres"
    networks:
      - ada_app_image-network

  postgres:
    container_name: 'postgres_ada'
    image: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: Nome_DB
    networks:
      - ada_app_image-network
    volumes:
      - db-data:/var/lib/postgresql/data

networks:
  ada_app_image-network:
    driver: bridge

volumes:
  db-data:
```

Rodei o Projeto com:

```bash
docker compose up -d
```

## Objetivo

Fixar conteúdo de Docker

## Como executar

 1. Instalar o Docker e Docker compose (O Docker Desktop inclui os dois)
 2. Copiar o repositório
 3. Iniciar um terminal na pasta onde tem o compose.ymal
 4. Rodar o comando: docker compose up -d
 5. Acessar o localhost:5000
 6. Clicar no Registar na barra de navegação (a primeira vez terá que aceitar rodar as migrations)

*Como o volume não será recriado a cada docker compose up -d então mesmo que você desça o container o banco de dados continua no volume

## Referências

[Instrutora - Thayse Frankenberger](https://www.linkedin.com/in/thayse-frankenberger-9832161b7/?originalSubdomain=br)

[Guia 1](https://medium.com/@kevinmwita7/create-a-asp-net-mvc-web-app-with-authentication-using-postgresql-as-its-database-3735355cbf9b)

[Guia 2](https://www.docker.com/blog/building-multi-container-net-app-using-docker-desktop/)

[Guia 3](https://github.com/docker/awesome-compose?tab=readme-ov-file)

## Autora

**Denize**

It is not luck, it is hard work!

<img style="border-radius: 50%;" src="https://user-images.githubusercontent.com/46844031/163518939-915f6e15-200a-4e9c-9f54-9bee6beec89b.jpg" width="100px;" alt=""/>

Where to find me:


[![Twitter Badge](https://img.shields.io/badge/Twitter-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white)](https://twitter.com/Dbassi91)   
[![Linkedin Badge](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/dbfigueiredo/)   
[![Gmail Badge](	https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:denize.f.bassi@gmail.com)   
[![CodePen](https://img.shields.io/badge/Codepen-000000?style=for-the-badge&logo=codepen&logoColor=white)](https://codepen.io/debafig)   
[![Facebook Badge](https://img.shields.io/badge/Facebook-1877F2?style=for-the-badge&logo=facebook&logoColor=white)](https://www.facebook.com/d.bassi91/)   
[![GitHub Badge](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/DeBaFig)   
[![Instagram Badge](https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white)](https://www.instagram.com/bassidenize/)   
[![About.me Badge](https://img.shields.io/badge/website-000000?style=for-the-badge&logo=About.me&logoColor=white)](https://debafig.github.io/me/)   
[![Whatsapp](https://img.shields.io/badge/WhatsApp-25D366?style=for-the-badge&logo=whatsapp&logoColor=white)](https://whatsa.me/5547935051914)
[![Discord](https://img.shields.io/badge/DeBaFig%235875-%237289DA.svg?style=for-the-badge&logo=discord&logoColor=white)](https://discordapp.com/users/DeBaFig#5875)
# Section 01: Initial the project via .NET CLI

This README documents the initial setup of the project, how to run it, and how to test the default endpoint using the REST Client extension. Further sections will document additional features and components as they are added to the project.

## 1.Requirements

- **IDE** (vscode or visual studio)
- **dotnet-sdk** (you can install in linux-arch via `sudo pacman -S dotnet-sdk` command.)
- **asp-runtime** (you can install in linux arch via `sudo pacman -S aspnet-runtime` command)

## 2. Project Initialization

```bash
dotnet new webapi -n Restaurants.API -controllers
```

This command does the following:

- `dotnet new webapi` : Creates a new Web API project.
- `-n Restaurants.API` : Specifies the name of the project.
- `-controllers` : Whether to use controllers instead of minimal APIs. Entering `dotnet new webapi` without specifying either option creates a minimal API project.

### 2.1 Add Solution File

After initializing the project, I created a solution file to manage the project using the following command:

```bash
dotnet new sln -n Restaurants
dotnet sln add ./Restaurants.API
```

This creates a solution file named `Restaurants.sln` and adds the `Restaurants.API` project to the solution.

## 3.Running the Application

### 3.1 Build and Run

```bash
dotnet run --project Restaurants.API
```

or

```bash
cd Restaurant.API/
dotnet run
```

This command starts the application, allowing it to serve HTTP requests.

### 3.2 Watching the changes

To run the application with file watching enabled (automatically restarting the application when code changes are detected), I used:

```bash
dotnet watch --project Restaurants.API
```

## 4. Testing the weatherForecast endpoint

ASP.NET Core Web API projects come with a default endpoint for testing purposes. To test this endpoint, I used the REST Client extension in Visual Studio Code.

### 4.1 Installing Rest Client

By default it is available in visual studio but for vscode you have to download and install the extension `REST Client` by `Huachao Mao`.

### 4.2 Creating a Request

By default dotnet created a `.http` file to test the file. Inside of that file there is a endpoint to test the `WeatherForecast` endpoint:

```http
@Restaurants.API_HostAddress = http://localhost:5177

GET {{Restaurants.API_HostAddress}}/weatherforecast/
Accept: application/json

###
```

### 4.3 Sending a Request

With the .http file created, I used the _**Send Request**_ button provided by the REST Client extension to execute the request and view the response.

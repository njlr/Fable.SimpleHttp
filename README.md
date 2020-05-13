# Fable.SimpleHttp [![Build Status](https://travis-ci.org/Zaid-Ajaj/Fable.SimpleHttp.svg?branch=master)](https://travis-ci.org/Zaid-Ajaj/Fable.SimpleHttp) [![Build status](https://ci.appveyor.com/api/projects/status/fgbd40ahcyrec5uw?svg=true)](https://ci.appveyor.com/project/Zaid-Ajaj/fable-simplehttp) [![Nuget](https://img.shields.io/nuget/v/Fable.SimpleHttp.svg?maxAge=0&colorB=brightgreen)](https://www.nuget.org/packages/Fable.SimpleHttp)

A library for easily working with Http in Fable projects.

### Features

 - Extremely simple API for working with HTTP requests and responses.
 - Implemented in idiomatic F# Async (instead of promises which follow JS semantics)
 - Supports sending and receiving raw binary data (i.e.`Blob` in the browser)
 - Built on top of `XMLHttpRequest` available in all browsers (even IE11!) so it doesn't need the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) nor it's associated polyfill.

### Installation
Install from nuget using paket
```sh
paket add nuget Fable.SimpleHttp --project path/to/YourProject.fsproj
```

## Usage
```fs
open Fable.SimpleHttp

// Functions from the Http module are all safe and do not throw exceptions

// GET request

async {
    let! (statusCode, responseText) = Http.get "/api/data"

    match statusCode with
    | 200 -> printfn "Everything is fine => %s" responseText
    | _ -> printfn "Status %d => %s" statusCode responseText
}

// POST request

async {
    let requestData = "{ \"id\": 1 }"
    let! (statusCode, responseText) = Http.post "/api/echo" requestData
    printfn "Server responded => %s" responseText
}

// Fully configurable request
async {
    let! response =
        Http.request "/api/data"
        |> Http.method POST
        |> Http.content (BodyContent.Text "{ }")
        |> Http.header (Headers.contentType "application/json")
        |> Http.header (Headers.authorization "Bearer <token>")
        |> Http.send

    printfn "Status: %d" response.statusCode
    printfn "Content: %s" response.responseText

    // response headers are lower cased
    response.responseHeaders
    |> Map.tryFind "content-length"
    |> Option.map int
    |> Option.iter (printfn "Content length: %d")
}


// Sending form data
async {
    let formData =
        FormData.create()
        |> FormData.append "firstName" "Zaid"
        |> FormData.append "lastName" "Ajaj"

    let! response =
        Http.request "/api/echo-form"
        |> Http.method POST
        |> Http.content (BodyContent.Form formData)
        |> Http.send

    printfn "Status => %d" response.statusCode
}


// Send and receive binary data with Blobs
// use FileReader module
async {
    let blob = Blob.fromText "input data"

    let! response =
       Http.request "/api/echoBinary"
       |> Http.method POST
       |> Http.content (BodyContent.Binary blob)
       |> Http.overrideResponseType ResponseTypes.Blob
       |> Http.send

    match response.content with
    | ResponseContent.Blob content ->
        let! blobContent = FileReader.readBlobAsText content
        printfn "Received content: %s" blobContent // "Received content: input data"

    | _ ->
        printfn "Unexpected response content"
}
```

## Building and running tests
Requirements

 - Dotnet core 2.1+
 - Mono 5.0+
 - Node 10.0+


Watch mode: both client and server live
```sh
./build.sh Start
```
Running the the end-to-end tests
```sh
./build.sh Test
```
or just `Ctrl + Shift + B` to run the cli tests as a VS Code task

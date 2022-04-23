# Static Blazor WASM L2T CORS Error

**Config:**

* L2T v6.14.0
* (ASP).NET 7 (Core) preview 3 SDK

**Error**

`Access to fetch at 'https://api.twitter.com/2/oauth2/token' from origin 'https://127.0.0.1' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.`

(*more info at the end ...*)

**Description**

This is issue occurs with a similar static Blazor WASM SPA project as I mentioned in issue [283](https://github.com/JoeMayo/LinqToTwitter/issues/283)

In this project I've added the L2T library to the client as well*. The client (*the browser*) is making the requests directly to the Twitter v2 API.

*) Because adding `LinqToTwitter.AspNet` library to a project such as this gives a build error (*see issue: [283](https://github.com/JoeMayo/LinqToTwitter/issues/283)*) I'm using the `LinqToTwitter` library here. Maybe that's why this error occurs?

It's possible that I'm doing something wrong but I file this issue because when you take a look at issue [283](https://github.com/JoeMayo/LinqToTwitter/issues/283) maybe you could take a look at this one too. It's the same configuration. 

**To Reproduce**

I've created a GitHub repo for this issue [here](https://github.com/JeepNL/L2T_CORS_Error) which you can download & run

* Twitter Project Settings / User authentication settings:

  * OAuth 2.0: True / On
  * Type of App: Single page App
  * Callback URI / Redirect URL = `https://127.0.0.1/OAuth2/Complete`;

* In [/Pages/Login.razor](https://github.com/JeepNL/L2T_CORS_Error/blob/master/L2T_CORS_Error/Pages/Login.razor)
  * Fill in your `clientId` (_from Twitter_)

**The Begin / Complete authentication process**

I do this a bit different than in your examples, that is: the 'Begin' part. I configure the URL to authenticate with Twitter in [/Pages/Login.razor](https://github.com/JeepNL/L2T_CORS_Error/blob/master/L2T_CORS_Error/Pages/Login.razor). This works, when you click the 'Login with Twitter' button you'll be redirected to the Twitter authentication page.

When returning from Twitter to [/Pages/OAuth2Complete.razor](https://github.com/JeepNL/L2T_CORS_Error/blob/master/L2T_CORS_Error/Pages/OAuth2Complete.razor) (*= `https://127.0.0.1/OAuth2/Complete`*) I'm getting the CORS error at line 43:

* `await auth.CompleteAuthorizeAsync(code, state);`

Because all of the code is running in the user's browser, you can't configure CORS here. CORS is configured at the server API which in this case is the Twitter API v2. Looking at the error I think there's an extra header needed in the request. I tried to find a solution for this but I couldn't find one. 

**Different Blazor (WASM) Projects**

I don't have this problem in the other Blazor WASM L2T project I'm working on, because  in that project the client (*the browser*) contacts my server API which then contacts the Twitter v2 API. 

In this new project the client contacts the Twitter API v2 directly. This is a static Blazor WASM client, which you can deploy on GitHub Pages, Azure Static Web Apps & Cloudflare Pages. These are all free hosting solutions and with Cloudflare Pages there isn't a bandwidth limit too. That's why this setup (a static client) is so interesting IMO.

I've created 2 different projects: 1) A Hosted Blazor WASM App & 2) A Static Blazor WASM App and when these are ready we can add them to your [LinqToTwitter/Samples/LinqToTwitter6/](https://github.com/JoeMayo/LinqToTwitter/tree/main/Samples/LinqToTwitter6) GitHub Samples directory.

There are 3 different Blazor configurations:

* Server Side Blazor (Not WASM, you have already a sample made by Michael Washington )
* Hosted Client Side Blazor WASM *(= Static Client Side Blazor + (server/hosted) Web API)*
* Static Client Side Blazor WASM

When I first started with L2T & Blazor WASM I thought adding the L2T Library to the client wasn't a good idea, but I was wrong. Because in many cases it makes more sense to contact the Twitter API from the client's browser directly. With some streaming scenarios it's a different story, having an inbetween server (web API) can be useful.



**1a) Text Dev Console Error**
```
Access to fetch at 'https://api.twitter.com/2/oauth2/token' from origin 'https://127.0.0.1' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

POST https://api.twitter.com/2/oauth2/token net::ERR_FAILED 400

crit: Microsoft.AspNetCore.Components.WebAssembly.Rendering.WebAssemblyRenderer[100]
Unhandled exception rendering component: TypeError: Failed to fetch
System.Net.Http.HttpRequestException: TypeError: Failed to fetch
 ---> System.Runtime.InteropServices.JavaScript.JSException: TypeError: Failed to fetch
   at System.Net.Http.BrowserHttpHandler.SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
   --- End of inner exception stack trace ---
   at System.Net.Http.BrowserHttpHandler.SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
   at System.Net.Http.HttpClient.<SendAsync>g__Core|83_0(HttpRequestMessage request, HttpCompletionOption completionOption, CancellationTokenSource cts, Boolean disposeCts, CancellationTokenSource pendingRequestsCts, CancellationToken originalCancellationToken)
   at LinqToTwitter.OAuth.OAuth2Authorizer.SendHttpAsync(HttpRequestMessage req)
   at LinqToTwitter.OAuth.OAuth2Authorizer.GetAccessTokenAsync(String code)
   at LinqToTwitter.OAuth.OAuth2Authorizer.CompleteAuthorizeAsync(String code, String state)
   at L2T_CORS_Error.Pages.OAuth2Complete.OnInitializedAsync() in D:\Repos\L2T_CORS_Error\L2T_CORS_Error\Pages\OAuth2Complete.razor:line 43
   at Microsoft.AspNetCore.Components.ComponentBase.RunInitAndSetParametersAsync()
   at Microsoft.AspNetCore.Components.RenderTree.Renderer.GetErrorHandledTask(Task taskToHandle, ComponentState owningComponentState)
```

**1b) Screenshot of Dev Console Error**

![Screenshot](https://github.com/JeepNL/L2T_CORS_Error/raw/master/L2T_CORS_Console.jpg)

**2a) Text Dev Console Request/Response Headers**

```
General

Request URL: https://api.twitter.com/2/oauth2/token
Request Method: POST
Status Code: 400 
Referrer Policy: strict-origin-when-cross-origin

Response Headers

cache-control: no-cache, no-store, must-revalidate, pre-check=0, post-check=0
content-disposition: attachment; filename=json.json
content-encoding: gzip
content-length: 119
content-type: application/json;charset=UTF-8
date: Thu, 21 Apr 2022 18:20:50 GMT
expires: Tue, 31 Mar 1981 05:00:00 GMT
last-modified: Thu, 21 Apr 2022 18:20:51 GMT
pragma: no-cache
server: tsa_o
set-cookie: guest_id=v1%3A165056525108332651; Max-Age=34214400; Expires=Mon, 22 May 2023 18:20:51 GMT; Path=/; Domain=.twitter.com; Secure; SameSite=None
strict-transport-security: max-age=631138519
x-connection-hash: 751182134f5d5d7c35fa1f6dd10802838a0c93b74409bb21bc8fc75b0417c175
x-content-type-options: nosniff
x-frame-options: SAMEORIGIN
x-response-time: 214
x-xss-protection: 0

Request Headers

:authority: api.twitter.com
:method: POST
:path: /2/oauth2/token
:scheme: https
accept: */*
accept-encoding: gzip, deflate, br
accept-language: en-US,en;q=0.9,nl;q=0.8
content-length: 324
content-type: application/x-www-form-urlencoded; charset=utf-8
dnt: 1
origin: https://127.0.0.1
referer: https://127.0.0.1/
sec-ch-ua: " Not A;Brand";v="99", "Chromium";v="102", "Microsoft Edge";v="102"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Windows"
sec-fetch-dest: empty
sec-fetch-mode: cors
sec-fetch-site: cross-site
user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.4976.0 Safari/537.36 Edg/102.0.1227.0
```

**2b) Screenshot of Dev Console Request/Response Headers**

![Screenshot](https://github.com/JeepNL/L2T_CORS_Error/raw/master/L2T_CORS_Devtools.jpg)



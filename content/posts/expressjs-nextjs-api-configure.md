+++
date = '2025-03-09T01:05:10+03:00'
draft = false
title = "NextJS & ExpressJS Local Network Device Settings"
+++

Hey there! When you use ExpresJS with NextJS application, most often you use default 'localhost' for development
proccess and while you use 'localhost', everything seem flawless at the beginning but when you tried to test this
application on your smartphone, you encounter with Network Error. Let's inspect this error and generate a solution for
it.

## Why this problem occurs?

The most basic reason for this problem is that localhost, that is, the IP Address '127.0.0.1', responds only to
connections from the computer it is working on. If you try to connect to your computer via your phone, this connection
will be via your computer's IP Address, not via localhost.

Without realizing this, giving the base url for the API as 'localhost' will result in the application not working on
mobile or even any device on the local network due to the API.

## Solutions

There are 2 ways to solve this problem.

The first and simplest way is to run the API at the IP Address '0.0.0.0', if you follow this way, your API will work
by accepting all IP Addresses.

For example;

```
app.listen(3000, '0.0.0.0', (error) => {
  if (!error) {
    console.log(`Server started; http://0.0.0.0:3000/`);
  }
});
```

The second is that, as I will explain below, it broadcasts by finding the IP Address of the device it is working on
every time and uses the appropriate API address for this broadcast.

Implementing the first solution is definitely less secure because it provides services regardless of which internet
interface it receives requests from, and this even allows you to expose your service. It is
recommended to use it in virtual network scenarios when you really know what you are doing.

I reached the second solution by thinking about the following ways;

- I needed to make an API call using my IP Address during the development phase via my NextJS application,
  I was going to set a Base API url for this, and it had to be updated every time my IP Address changed.

- Since I will use this server and client on my own internet network, ExpressJS will run on my server on my local
  network, so it has the same IP Address (both client and server were running on the same device). For this reason, it
  had to broadcast on the same IP Address on the ExpressJS server.

[You can access the 'os' module page here.](https://www.npmjs.com/package/os)

ExpressJS / app.js;

``` 
const os = require('os');
const networkInterfaces = os.networkInterfaces();

const PORT = 5693;

// If you have any WSL or other connections that may be visible under the
// 'Wi-Fi' section, you have to change this value to another index (1,2,3 etc...)

const HOST = networkInterfaces["Wi-Fi"][0].address;

app.listen(PORT, HOST, (error) => {
  if (!error) {
    console.log(`Server started: http://${HOST}:${PORT}/`);
  }
});
```

In this way, you can configure your ExpressJS server to broadcast on the IP Address of the device it is running on. Now
we need to update the settings of our NextJS application. NextJS already comes with built-in .env support (NextJS 9.4
version, 2020), so we are going to define our variables 'next.config.js' instead of .env file we usually do.

NextJS / next.config.js;

```
/** @type {import('next').NextConfig} */
import os from 'os';
const wifiInterface = os.networkInterfaces();
const apiAdress = wifiInterface["Wi-Fi"][0].address

const nextConfig = {
  reactStrictMode: true,
  env: {
    API_BASE: apiAdress ? `http://${apiAdress}:5693/api` : 'http://localhost:5693/api'
  }
};

export default nextConfig;
```

After making all of these configurations, your NextJS application and ExpressJS API both working on the same IP Adress.

## Final

With this process, I got rid of having to update my IP Address from my .env file every time my IP Address changes. In
the same way, I also got rid of the Network Error that appeared on other devices on the network when trying to connect
to the API as a 'localhost'. I hope you found my sharing useful.

**\-The links and the apps I share are just for sharing information. I don’t have any ads or partnerships.\-**

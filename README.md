<p align="center">
    <img src="https://user-images.githubusercontent.com/6702424/154800287-c8433ac4-26c1-43fb-9507-46cd0b6e751a.png">  
</p>
<p align="center">
    <i>Bundle your environment variable in your create-react-app at <strike>build time</strike> <b> launch time</b>!</i>
    <br>
    <br>
    <a href="https://github.com/garronej/cra-envs/actions">
      <img src="https://github.com/garronej/cra-envs/workflows/ci/badge.svg?branch=main">
    </a>
    <a href="https://bundlephobia.com/package/cra-envs">
      <img src="https://img.shields.io/bundlephobia/minzip/cra-envs">
    </a>
    <a href="https://github.com/garronej/cra-envs/blob/aa97a3cc446a0afdb7769a1d351c5b45723d3481/tsconfig.json#L14">
        <img src="https://camo.githubusercontent.com/0f9fcc0ac1b8617ad4989364f60f78b2d6b32985ad6a508f215f14d8f897b8d3/68747470733a2f2f62616467656e2e6e65742f62616467652f547970655363726970742f7374726963742532302546302539462539322541412f626c7565">
    </a>
    <a href="https://github.com/garronej/cra-envs/blob/main/LICENSE">
      <img src="https://img.shields.io/npm/l/cra-envs">
    </a>
</p>

# Motivation

Create-react-app [supports environment variable](https://create-react-app.dev/docs/adding-custom-environment-variables/) but they are bundled at build time when `yarn build` is run.  
If we want to change anything like the URL of the backend the app should connect to, we have to rebuild, we can't ship customizable Docker image of our CRA apps.  

The solution would be to be able to do:  
```bash
 docker run --env FOO="xyz" my-org/my-create-react-app
 ```
Then access `FOO`:  
- In the code like `process.env["FOO"]` 
- In `public/index.html` like `<title>%FOO%</title>`

`cra-envs` does just that, in a secure, performant and type safe way.  

# Features

- ✅ No impact on the startup time.
- ✅ No impact on the Docker image size.  
- ✅ Require no network connection at container startups.
- ✅ Secure: It only injects the envs declared in the `.env` file.  
- ✅ It works like you are already used to. It just changes **when** the envs
- ✅  EJS support in `public/index.html` ([few peoples knows](https://github.com/facebook/create-react-app/issues/3112#issuecomment-328829771)).  
This enables for example to conditionally [preload one font or another](https://github.com/garronej/cra-envs-demo-app/blob/e1aa8067b52a563bc5db18558e7ed7746a56c9c0/public/index.html#L6-L21)  
- ✅ (Optional) Type safe: An env getter is generated so [you know what envs are available](https://user-images.githubusercontent.com/6702424/154802407-92d2d0b7-b74c-4b35-a2b5-5c27c26d5127.png) are injected.  


# Usecase example

<p align="center">
	<img src="https://user-images.githubusercontent.com/6702424/147029706-40b2d807-2733-44fe-9f59-08b935555848.png" />
</p>

[Onyxia-web](https://github.com/InseeFrLab/onyxia-web) is a create-react-app distributed as a [Docker image](https://hub.docker.com/r/inseefrlab/onyxia-web/tags).  

Sysadmins that would like to deploy Onyxia on their infrastructure can simply use
[the official Docker image](https://hub.docker.com/r/inseefrlab/onyxia-web/tags) and provide relevant environnement variable to adust the theme/branding of the website to their usecase.  

Deployment examples:  
- [SSPCloud Datalab](https://datalab.sspcloud.fr)
- [SILL test bench](https://garronej.dev)  


<p align="center">
  <a href="https://datalab.sspcloud.fr">
    <img src="https://user-images.githubusercontent.com/6702424/154808798-8f5c29d0-cf39-4b34-96d0-ba0388b52037.png">  
  </a>
  <a href="https://garronej.dev">
    <img src="https://user-images.githubusercontent.com/6702424/154808797-fd2b6152-e6ac-4bea-9f18-53617f07581c.png">  
  </a>
</p>

# Documentation

Find 👉[**here**](https://github.com/garronej/cra-envs-demo-app)👈 a demo setup of:  
`cra-envs` + `create-react-app` + `TypeScript` + `Nginx` + `Docker`


# More details on how it works

The recommended way to get started with `cra-envs` is to follow the instructions
provided in [the cra-envs-demo-app](https://github.com/garronej/cra-envs-demo-app).  
Now, if you want to acquire a deeper understanding what the tool does and how
you can follow the following steps.

Start by installing the tool: 

```bash
yarn add cra-envs 
```

Then declare all the allowed environment variables into the `.env` file of your project

Example
```ini
REACT_APP_FOO="Default value of foo"
REACT_APP_BAR=
REACT_APP_BAZ=
REACT_APP_FIZZ=
```

Once it's done run the script `npx generate-env-getter` ( Use `npx generate-env-getter js` if you your project don't use TypeScript)

It will generate `src/env.ts` ( or `src/env.js`) looking like:
```typescript
/* 
* Automatically generated by cra-envs.
* If you wish to declare a new environment variable declare it in the .env file (prefixed by REACT_APP_)
* then run 'npx generate-env-getter' at the root of your project.
* This file will be updated.
*/
import { getEnvVarValue } from "cra-envs";

export const envNames = [
  "FOO",
  "BAR",
  "BAZ",
  "FIZZ"
] as const;

export type EnvNames = typeof envNames[number];

let env: Record<EnvNames, string> | undefined = undefined;

export function getEnv() {

    if (env === undefined) {
        env = {} as Record<EnvNames, string>;
        for (const envName of envNames) {
            env[envName] = getEnvVarValue(envName);
        }
    }

    return env;

}
```
(This file should be gitignored)  

Now let's test it by creating a `.env.local` file like:  
```ini
REACT_APP_BAR="Value of bar defined in .env.local"
```

And let's do this somewhere in our code: 

```typescript
import { getEnv } from "./env.ts"

console.log(getEnv());
```
Now if we run `REACT_APP_BAZ="Value of baz passed inline" yarn start` we get this
in the console: 

```json
{
    "FOO": "Default value of foo",
    "BAR": "Value of bar defined in .env.local",
    "BAZ": "Value of baz passed inline",
    "FIZZ": ""
}
```

Now if you run `yarn build` then `BAZ="Value of baz on the server" npx embed-environnement-variables`
the value of `BAZ` will be injected in `build/index.html` (or `html/index.html`) so that if you 
start statically serving
the `build/` dir, for example with `serve -s build` you will get this in the console:  

```json
{
    "FOO": "Default value of foo",
    "BAR": "Value of baz on the server",
    "BAZ": "",
    "FIZZ": ""
}
```

Note that on the server the environment variable names don't need to be prefixed with `REACT_APP_` (they can though).
Also note that the script runs very fast and thus represent virtually no overhead when starting your container.  
By default `embed-environnement-variables` does not embed variables defined in `.env.local`, if you want to include
them use: `--includes-.env.local` or `-i`.

The next step is to set up a clean Dockerfile where there is both node and Ngnix available.  
Node for being able to run `npx embed-environnement-variables` and Ngnix for serving the app.  
It is also important to make sure `cra-envs` is not bootstrapped by `npx` in the entrypoint.

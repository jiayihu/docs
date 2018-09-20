import withDoc from '../../../lib/with-doc'

import Now from '../../../components/now/now'
import { GenericLink } from '../../../components/text/link'
import { InlineCode } from '../../../components/text/code'
import { TerminalInput } from '../../../components/text/terminal'
import Caption from '../../../components/text/caption'

export const meta = {
  title: 'Building and Deploying a Single Page Application with Create React App',
  description: 'Creating a single page web app with Create React App and deploying it with Now',
  date: '08 September 2018',
  editUrl: 'pages/docs/examples/create-react-app.md',
  image: IMAGE_ASSETS_URL + '/docs/examples/create-react-app/twitter-card.png'
}

[Create React App](https://github.com/facebookincubator/create-react-app) (CRA) is a boilerplate tool used to create static single-page [React](https://reactjs.org/) applications without build configuration.

The focus of this page will be on how to deploy an application built with Create React app.

## Get Started with Create React App
To get started, use npm's `npx` command to run the `create-react-app` module which will create a React app in a specified directory. For example, the following command, when entered in a terminal with npm installed, will create a React application within the `my-app` directory:
<TerminalInput>
npx create-react-app my-app
</TerminalInput>
<Caption>Running `create-react-app` to build an application in the `my-app` directory with `npx`.</Caption>

With an initialized application in the `my-app` directory, Create React App has provided a ready-made environment to pick up and go.

Built-in commands provide easy development and production tools, such as running `yarn start` for a local development server, `yarn test` for a test-runner, or `yarn build` to export the app statically.

For alternative methods of installing and a more in-depth guide for how Create React App can help to get started quickly with React, read [the documentation](https://github.com/facebook/create-react-app#creating-an-app) for it.

With a freshly created React app, and any changes to is after, it is now time to deploy the app.

## Deploying the App with Now
For production builds, Create React App provides a command `build`, within the `package.json` file, that exports the React application statically. Now supports [static applications](/docs/static-deployments/introduction-and-deploying) with no need for additional config. However, in the case of Create React App, it is possible to [build the app on Now](/docs/static-deployments/builds/building-with-now), as well as deploy it live.

### Preparing for Now
There are a few things to configure to best deploy a React application built with Create React App.

Firstly, the app is a Single Page Application. This means that the entire application will run from a single page, in this case, `index.html`. To help with this, Now supports a [`rewrites` configuration option](/docs/static-deployments/configuration#rewrites-(array)), allowing a method of redirecting users from one path to another.

This configuration option will go in a `now.json` configuration file under the `static` property. We will also set the `type` of deployment to `static` to let Now know that we want a static deployment with [a build step](#instructing-the-build).

```json
{
  "type": "static",
  "static": {
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  }
}
```
<Caption>A <InlineCode>now.json</InlineCode> configuration setting deployment type as <InlineCode>static</InlineCode> and enabling the rewriting of all paths to point to <InlineCode>index.html</InlineCode>.</Caption>

Setting the deployment type to `static` is essential due to the method of [building the app](#instructing-the-build) on Now.

Using a `Dockerfile` that contains build instructions, Now will be able to build the app on deployment, as well as running any tests, stopping the build if they fail.

### Instructing the Build
To build an application created with Create React App when deploying, create a `Dockerfile` containing the build instructions like the following:

```
# Use Node.js version 10
FROM mhart/alpine-node:10

# Set the working directory
WORKDIR /usr/src

# Copy package manager files to the working directory and run install
COPY package.json yarn.lock ./
RUN yarn install

# Copy all files to the working directory
COPY . .

# Run tests
RUN CI=true yarn test

# Build the app and move the resulting build to the `/public` directory
RUN yarn build
RUN mv ./build /public
```
<Caption>A <InlineCode>Dockerfile</InlineCode> containing the build instructions for a React app created with Create React App.</Caption>

Additionally, we can whitelist specific files or directories so that no unnecessary files are deployed with the build, using a `.dockerignore` file:

```
*
!src
!public
!package.json
!yarn.lock
```
<Caption>A <InlineCode>.dockerignore</InlineCode> file that is whitelisting the <InlineCode>src</InlineCode> and <InlineCode>public</InlineCode> directories and package manager files.</Caption>


### Deploying
With the configuration complete, when deploying, Now will recognize the options passed in and build the app as expected. All that is left to do is deploy:
<TerminalInput>now</TerminalInput>

export default withDoc({...meta})(({children}) => <>{children}</>)

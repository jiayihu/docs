import asset from 'next/asset'
import withDoc from '../../../lib/with-doc'

import Now from '../../../components/now/now'
import { GenericLink } from '../../../components/text/link'
import { InlineCode } from '../../../components/text/code'
import Caption from '../../../components/text/caption'
import Image from '../../../components/image'
import { TerminalInput } from '../../../components/text/terminal'

export const meta = {
  title: 'Building and Deploying a Next.js Application',
  description: 'Get started building and deploying a basic Next.js application with Now',
  date: '10 September 2018',
  editUrl: 'pages/docs/examples/next.md',
  image: IMAGE_ASSETS_URL + '/docs/examples/next/twitter-card.png'
}

[Next.js](http://nextjs.org) is a lightweight framework for creating static and server-rendered applications, which abstracts away all of the complicated parts required for creating a [React](http://reactjs.com/) application.

## Get Started with Next.js
To start, create a directory for the Next.js application to go into:
<TerminalInput>mkdir next-app && cd next-app</TerminalInput>

In order to use Next.js, we need to install Next itself, [React](https://facebook.github.io/react/), and [ReactDOM](https://facebook.github.io/react/docs/react-dom.html):

<TerminalInput>
yarn add next react react-dom
</TerminalInput>
<Caption>Installing <InlineCode>next</InlineCode>, <InlineCode>react</InlineCode>, and <InlineCode>react-dom</InlineCode> using <GenericLink href="https://yarnpkg.com">Yarn</GenericLink>.</Caption>

> Alternatively, use [npm](npmjs.org) to install these packages by using `npm install next react react-dom --save`.

Next, create the project's `package.json` in that directory:
```json
{
  "name": "next-app",
  "scripts": {
    "dev": "next",
    "build": "next build",
    "start": "next start"
  }
}
```
<Caption>The name and scripts content of a basic Next.js application within a <InlineCode>package.json</InlineCode> file.</Caption>

Alternatively, you can statically export a Next.js application by using an additional script within the `"scripts"` section:
```
"export": "next export"
```

### Adding Content

Now that the project's meta files are in place, code can be added that will be rendered when a visitor accesses the site.

With most frameworks, you would now have to set up a router and tell it about the page you would like to add. With [Next.js](https://nextjs.org), the only thing to do is create a page, and the framework will automatically handle the rest.

In turn, the next step will be to add a directory named "pages". As the name indicates, this is where all pages of your application will be located. Based on the directory structure, each file will be mounted on a specific path.

For example, add an `index.js` file inside the "pages" directory, which will be served when the `/` path is accessed on the app. Similarly, `about.js` will route to `/about`.

Into the `index.js` file, place the following code:
```
export default () => (
  <p>Welcome to Next.js!</p>
)
```

The above code is exporting [JSX](https://facebook.github.io/react/docs/jsx-in-depth.html) which renders a paragraph with text content inside of it.

Next.js is now ready to serve content.

### Trying out the Code

[Next.js](https://nextjs.org) comes with an excellent development toolchain built-in. We've defined one of these commands [earlier](#get-started-with-next.js) in the `package.json` file we've created: The `dev` script (linked to the `next` command), which can be can run like this:

<TerminalInput>npm run dev</TerminalInput>

When running the command, Next.js will build the code and serve it on the address shown in the terminal. Also, it will watch for changes on the files inside your project. If you make changes to the code, the app will be built again.

This is how it should look:

<Image
  src={asset(`${IMAGE_ASSETS_URL}/docs/next/running.png`)}
  width="550"
  height="309"
/>

You can now go to <http://localhost:3000> in your browser and see the rendered markup:

<Image
  src={asset(`${IMAGE_ASSETS_URL}/docs/next/output.png`)}
  width="550"
  height="307"
/>

Now the application is **ready to be deployed**.

For more information on Next.js, visit [the website](https://nextjs.org) and [the documentation](https://nextjs.org).

## Deploying the App
There are two methods for deploying a Next.js application due to the ability of Next being able to both render server-side and statically at build-time.

### Deploying Statically
For statically exported Next.js applications using `next export` in the package.json scripts, as noted in the [Getting Started](#get-started-with-next.js) portion of this example guide, Now supports the static export by default.

[Static Builds on Now](/docs/static-deployments/builds/building-with-now) allow Next.js apps to be built and deployed all with one command.

### Building the App on Now
To build the app when deploying, first set the type of deployment to `static` within a `now.json` configuration file:
```json
{
  "type": "static"
}
```

With the deployment type set as static, a Dockerfile can be created and Now will detect the Dockerfile and take the instructions within as the build steps.

Using a Node base image, the Dockerfile can use the scripts within the `package.json` to build and export the Next.js app statically into the `public` directory, which is where Now will use to serve the content from.

```
# Using lightweight Node.js base with Yarn to install our dependencies and build and export the app
FROM mhart/alpine-node:10

# Set the working directory and copy over package manager specific files
WORKDIR /usr/src
COPY package.json yarn.lock ./

# Install dependencies
RUN yarn

# Copy the rest of the app files to the working directory
COPY . .

# Build the app and export the static files to the `public` directory
RUN yarn build && yarn export -o /public
```
<Caption>A <InlineCode>Dockerfile</InlineCode> containing the build instructions for a static Next.js application.</Caption>

Additionally, create a `.dockerignore` file to whitelist the files needed to avoid deploying anything unnecessary:
```
*
!pages
!components
!package.json
!yarn.lock
```
<Caption>A <InlineCode>.dockerignore</InlineCode> file whitelisting necessary directories and files for deployment.</Caption>

With the configuration set and the build instructions defined, the app is ready to be deployed and built with just one command:
<TerminalInput>now</TerminalInput>

The app will be then built and deployed with Now, providing you with a deployment link to share or to [alias](/docs/features/aliases) for staging or to push live.

[View the example application on the zeit/now-examples repository.](https://github.com/zeit/now-examples/tree/master/static-next.js)

### Deploying a Server-Rendered Next.js App
To deploy a server-rendered Next.js application, all that's needed is to set the deployment type and create instructions for how Now can run the app within the deployment.

Set the deployment type to `docker` and enable Now Cloud v2 within a `now.json` file:
```json
{
  "type": "docker",
  "feature": {
    "cloud": "v2"
  }
}
```

Then, create a `Dockerfile` with the instructions to prepare and start the server that will render the Next.js application:
```
# Use a Node.js image and assign it as our build
FROM mhart/alpine-node:10 as build

# Set the working directory, copy dependency management files to the working directory,
# and install the dependencies
WORKDIR /usr/src
COPY package.json yarn.lock ./
RUN yarn install

# Copy all files to the working directly, build the application
# and purge the development dependencies
COPY . .
RUN yarn build && yarn --production

# Create a new image using a minimal Node.js image
# with no extra tools packaged in, such as Yarn or npm for the smallest final size
FROM mhart/alpine-node:base-10

# Set the working directory for the new image and
# set the `NODE_ENV` environment variable value to `production`
# along with setting the path for node_modules to be accessible
WORKDIR /usr/src
ENV NODE_ENV="production"
ENV PATH="./node_modules/.bin:$PATH"

# Copy files from the base image over to our new image's working directory
COPY --from=build /usr/src .

# Start the server for Next.js using Node.js
CMD ["next", "start"]
```
<Caption>A <InlineCode>Dockerfile</InlineCode> utilizing <GenericLink href="https://docs.docker.com/develop/develop-images/multistage-build/">multi-stage builds</GenericLink> to produce a minimal final image which, in combination with Now Cloud v2, helps with fast cold-boot times and performance.</Caption>

Additionally, with a `.dockerignore` file, we avoid deploying any unnecessary files:
```
*
!pages
!components
!package.json
!yarn.lock
```

Finally, all that is left to do is deploy:
<TerminalInput>now</TerminalInput>

With that, your app will be built and ran with Now, which will give you the resulting deployment link to share and push live.

[View the example application on the zeit/now-examples repository.](https://github.com/zeit/now-examples/tree/master/node-next.js)

export default withDoc({...meta})(({children}) => <>{children}</>)

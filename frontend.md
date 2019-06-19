# Frontend development at dictyBase
All frontend web applications are built using [React](https://facebook.github.io/react/). Below is a list of all tools and configuration to use when starting a new project.

## React web applications
+ **[Create-react-app](https://github.com/facebookincubator/create-react-app)** - Default tool for scaffolding a web application. Use this to get started immediately.

+ **[Linting with ESLint](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#displaying-lint-output-in-the-editor)** - To ensure that the code adheres to best practices. Our `.eslintrc` will be updated periodically as necessary, and this repo will store the [latest version](https://github.com/dictybase-playground/dev-notes/blob/master/.eslintrc).

+ **[Code formatting with Prettier](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#formatting-code-automatically)** - Integrating with `eslint` is not recommended, instead allow it run independently. It should also be integrated with your editor of choice. Add the following to your `package.json`:
  ```
  "lint-staged": {
    "src/**/*.{js,jsx,json,css}": [
      "prettier --no-semi --trailing-comma all --jsx-bracket-same-line true --write",
      "git add"
    ]
  },
  ```

+ **[Husky githooks](https://github.com/typicode/husky)** - Makes it easy to use githooks as if they are npm scripts. Add the following to `scripts` in `package.json`:
  ```
  "precommit": "lint-staged"
  ```
  Then add this to your `package.json` as well:
  ```
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  ```

+ **[.editorconfig](https://editorconfig.org/)** - Add our [.editorconfig](https://github.com/dictybase-playground/dev-notes/blob/master/.editorconfig) file into the root directory of your application. This helps maintain a consistent coding style between editors.

+ **[Semantic versioning](https://github.com/semantic-release/semantic-release)** - Install the following:
`npm i -D commitizen semantic-release @semantic-release/changelog @semantic-release/git`

  Add the following to `package.json`:
  ```
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  },
  "release": {
    "noCi": true,
    "tagFormat": "${version}",
    "verifyConditions": [
      "@semantic-release/changelog",
      "@semantic-release/git"
    ],
    "prepare": [
      "@semantic-release/changelog"
    ],
    "publish": [
      "@semantic-release/github"
    ]
  }
  ```

  And these into your `package.json` scripts:
  ```
  "cz": "git-cz",
  "semantic-release": "semantic-release"
  ```

+ **[Static type checking with Flow](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#adding-flow)** -  Use [flow-typed](https://github.com/flowtype/flow-typed) to add support for 3rd party libraries. Also make sure to ignore the
  `node_modules` folder in the `.flowconfig` file. Here's what a basic config file should look like:
     ```
        [ignore]
        <PROJECT_ROOT>/node_modules/*

        [include]

        [libs]

        [options]

        [lints]
    ```

    Modules lacking `flow-typed` definitions can added by creating a stub.
    ```
        flow-typed create-stub module@version
    ```

    Let flow ignore source code line by adding `// $FlowFixMe` on the top.

+ **[Component in isolation](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#developing-components-in-isolation)** - Use
  [styleguidist](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#getting-started-with-styleguidist)
  for that.

+ **Additional modules** - For styling and UI, install [Material-UI](https://material-ui.com/)
  ```
    npm i --save @material-ui/core
  ```
  
+ **Color scheme** - we have decided on a blue color scheme with the following colors (in order from dark to light):
  - #004080 (darkest, used in header and footer)
  - #0059b3 (halfway point between this and medium, which is kind of bright)
  - #0073e6 (medium)
  - #3399ff
  - #80c1ff
  - #cce6ff
  - #e6f2ff (lightest)

## Standalone React components

+ **[Using library mode of nwb](https://github.com/insin/nwb/blob/master/docs/guides/ReactComponents.md#developing-react-components-and-libraries-with-nwb)** - Default tool for scaffolding a standalone library component(s). The rest of the setup is tailored to this tool.

+ **Linting with ESLint setup**
  + [Replicate](https://github.com/facebookincubator/create-react-app/tree/v1.0.10/packages/eslint-config-react-app#usage-outside-of-create-react-app)
   the eslint setup of `create-react-app`. 
    ```
        npm i --save-dev eslint-config-react-app babel-eslint@7.2.3 eslint@3.19.0 eslint-plugin-flowtype@2.33.0 eslint-plugin-import@2.2.0 eslint-plugin-jsx-a11y@5.0.3 eslint-plugin-react@7.0.1
    ```
    Then add the following to `.eslintrc` file.
    ```
        { "extends": "react-app" }
    ```
  + To integrate with webpack(similar to `cra`), first install the following ...
    ```
        npm i --save-dev react-dev-utils eslint-loader@1.7.1
    ```

    Then add the following configuration in `nwb.config.js` file

    ```
        var eslintWebpack = {
          module: {
            rules: [
              {
                test: /\.(js|jsx)$/,
                enforce: "pre",
                use: [
                  {
                    options: {
                      formatter: require("react-dev-utils/eslintFormatter"),
                    },
                    loader: require.resolve("eslint-loader"),
                  },
                ],
                exclude: /node_modules/,
              },
            ],
          },
        }
    ```

    Then append the following to `module.exports`
    ```
          webpack: {
            extra: eslintWebpack,
          }
    ```

    It should look like this....

    ```
        module.exports = {
          type: "react-component",
          npm: {
            esModules: true,
            umd: false,
          },
          webpack: {
            extra: eslintWebpack,
          },
        }
    ```

+ **Code formatting with prettier** - Should be identical to the above setup with `cra`.

+ **Static type checking with flow** - Should be identical to the above setup with `cra`.

+ **Component in isolation** 
  + The initial [installation](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#getting-started-with-styleguidist) of the packages will be identical with `cra`. 
    ```
        npm i --save-dev react-styleguidist
    ```

    Then add scripts to `package.json`
    ```
        "styleguide": "styleguidist server",
        "styleguide:build": "styleguidist build"
    ```

  + Depending on the location, modify how to find the component files in `styleguide.config.js`. For example,
    ```
        module.exports = {
            components: "src/*.js",
        }
    ```
  + After that install the `cra` babel preset
    ```
        npm i --save-dev babel-preset-react-app 
    ```
    and add the following to `.babelrc`
    ```
        {"presets": ["react-app"]}
    ```
    Then add the following webpack configuration in `styleguide.config.js` right after the components section ...
    ```
      webpackConfig: {
        module: {
          rules: [
            {
              test: /\.jsx?$/,
              exclude: /node_modules/,
              loader: "babel-loader",
            },
            {
              test: /\.woff(2)?(\?v=[0-9]\.[0-9]\.[0-9])?$/,
              loader: "url-loader",
            },
            {
              test: /\.(ttf|eot|svg)(\?v=[0-9]\.[0-9]\.[0-9])?$/,
              loader: "file-loader",
            },
            {
              test: /\.(jpg|png)$/,
              exclude: /node_modules/,
              loader: "url-loader",
              options: {
                limit: 25000,
              },
            },
            {
              test: /\.css$/,
              use: [{ loader: "style-loader" }, { loader: "css-loader" }],
            },
          ],
        },
      },
   ```
+ **Unit testing with [jest](https://facebook.github.io/jest/)** 
  + Install required packages
    ```
        npm i --save-dev jest babel-jest react-test-renderer
    ```

  + Add `jest` to the scripts section of `package.json`
    ```
        "scripts": {
            "test": "jest"
        }
    ```

  + Add test files preferably in the same folder as the component. If possible, start with [snapshot
    testing](https://facebook.github.io/jest/docs/en/snapshot-testing.html#content)

  + Configure `jest` to manage static assets similar to the handling by webpack. First add the following to the `package.json`.
    ``` 
      // package.json
      
       "jest": {  
         "moduleNameMapper": {  
               "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/__mocks__/fileMock.js",  
               "\\.(css|less)$": "<rootDir>/__mocks__/styleMock.js"  
        }  
       },
    ```

  And then create two mock files

  > // __mocks__/styleMock.js  
  >         module.exports = {}

  > // __mocks__/fileMock.js  
  >         module.exports = 'test-file-stub'

  + Then run the test with
    ```npm test```

    Or watch and run tests on only changed files  
    ```npm test -- --watch```

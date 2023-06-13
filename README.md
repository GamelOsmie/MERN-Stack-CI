# MERN-Stack-CI
CI to run GitHub actions for my MERN stack apps

    name: MERN Stack CI

    on:
      push:
        branches:
          - '**'

    jobs:
      build:
        runs-on: ubuntu-latest

        strategy:
          matrix:
            node-version: [14.x, 16.x, 18.x]

        steps:
          - uses: actions/checkout@v3

          - name: Install Node.js ${{ matrix.node-version }}
            uses: actions/setup-node@v3
            with:
              node-version: ${{ matrix.node-version }}
              cache: 'npm'

          - name: Install server dependencies
            working-directory: ./App-name/server
            run: npm ci

          - name: Run server tests
            working-directory: ./App-name/server
            run: npm test

          - name: Install client dependencies
            working-directory: ./App-name/client
            run: npm ci

          - name: Run client tests
            working-directory: ./App-name/client
            run: npm test

          - name: Build client app
            working-directory: ./App-name/client
            run: npm run build

          - name: Copy client build to server public folder
            working-directory: ./App-name/client
            run: cp -R build/ ../server/public/


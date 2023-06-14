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

            - name: Update branch status
               if: always()
              run: |
                if [ ${{ job.status }} == 'success' ]; then
                  curl -X POST -H "Authorization: Bearer ${{ secrets.GITHUB_TOKEN }}" \
                    -H "Accept: application/vnd.github.v3+json" \
                    "https://api.github.com/repos/${{ github.repository }}/statuses/${{ github.sha }}" \
                    -d '{
                      "state": "success",
                      "target_url": "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}",
                      "description": "All tests passed",
                      "context": "CI Tests"
                    }'
                else
                  curl -X POST -H "Authorization: Bearer ${{ secrets.GITHUB_TOKEN }}" \
                    -H "Accept: application/vnd.github.v3+json" \
                    "https://api.github.com/repos/${{ github.repository }}/statuses/${{ github.sha }}" \
                    -d '{
                      "state": "failure",
                      "target_url": "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}",
                      "description": "Some tests failed",
                      "context": "CI Tests"
                    }'


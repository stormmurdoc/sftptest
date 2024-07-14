# SFTP File Download

This repository provides a workflow using GitHub Actions
that retrieves a file from a remote SFTP server and stores
it in the `data` directory within the repository.

## Code

```shell

name: SFTP File Download

on:
  schedule:
    - cron: '0 * * * *' # Runs every hour; adjust as needed
  workflow_dispatch: # Allows manual triggering of the workflow

jobs:
  download-file:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Install lftp
        run: sudo apt-get install -y lftp

      - name: Retrieve file via SFTP
        env:
          SFTP_SERVER: ${{ secrets.SFTP_SERVER }}
          SFTP_USERNAME: ${{ secrets.SFTP_USERNAME }}
          SFTP_PASSWORD: ${{ secrets.SFTP_PASSWORD }}
          REMOTE_FILE_PATH: ${{ secrets.REMOTE_FILE_PATH }}
          LOCAL_FILE_DIR: "./data"
        run: |
          TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
          FILE_NAME=$(basename $REMOTE_FILE_PATH)
          lftp -c "set sftp:connect-program 'ssh -a -x -o StrictHostKeyChecking=no'; open -u $SFTP_USERNAME,$SFTP_PASSWORD sftp://$SFTP_SERVER; get $REMOTE_FILE_PATH -o $LOCAL_FILE_DIR/${FILE_NAME%.*}_$TIMESTAMP.${FILE_NAME##*.}"

      - name: Commit and push file
        run: |
          git config --local user.email "github-actions[bot]@users.noreply.github.com"
          git config --local user.name "github-actions[bot]"
          git add $LOCAL_FILE_DIR/*
          git commit -m "Add file downloaded via SFTP on $(date +"%Y-%m-%d %H:%M:%S")"
          git push
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

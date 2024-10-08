# SFTP File Download

This repository provides a workflow using GitHub Actions
that retrieves a file from a remote SFTP server and stores
it in the `data` directory within the repository.

## Code

```shell

name: SFTP File Download

on:
  schedule:
    - cron: '00 8 * * *'  # Runs once a day
  workflow_dispatch:      # Allows manual triggyring of the workflow

jobs:
  download-file:
    runs-on: ubuntu-latest

    permissions:
      # Give the default GITHUB_TOKEN write permission to commit and push the
      # added or changed files to the repository.
      contents: write

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
          LOCAL_FILE_DIR: "data"
        run: |
          mkdir -p $LOCAL_FILE_DIR
          TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
          FILE_NAME=$(basename $REMOTE_FILE_PATH)
          lftp -c "set sftp:connect-program 'ssh -a -x -o StrictHostKeyChecking=no'; open -u $SFTP_USERNAME,$SFTP_PASSWORD sftp://$SFTP_SERVER; get $REMOTE_FILE_PATH -o $LOCAL_FILE_DIR/${FILE_NAME%.*}_$TIMESTAMP.${FILE_NAME##*.}"

      - name: Commit and push file
        run: |
          git config --local user.email "github-actions[bot]@users.noreply.github.com"
          git config --local user.name "github-actions[bot]"
          git add .
          git commit -m "Add file downloaded via SFTP on $(date +"%Y-%m-%d %H:%M:%S")"
          git push
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

```

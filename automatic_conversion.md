# Automatic Conversion of Microsoft Documents to Markdown

## Pandoc 

The conversion of Microsoft Documents to Markdown is achieved using Pandoc, a command-line tool and library for converting documents from one format to another. Pandoc can extract the text and images from the document and create the relevant markdown to format the new file to mimic the document that it was converted from. While these conversions are not always 100% accurate, they do mitigate the amount of work needed to transfer the information.

## GitHub Actions Workflow

```yml
name: Pandoc Conversion
on:
  push:
    branches:
      - development
```

This portion of the code means that on every push to the development branch, this workflow will activate.

```yml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install pandoc
        run: |
          sudo apt-get update
          sudo apt-get install -y pandoc
```

This is the initial setup of the virtual environment and installing the pandoc.

```yml
      - name: Convert and save media
        run: |
          find docs -type f -name "*.docx" -print0 | while IFS= read -r -d '' f; do
            filename=$(basename "$f" .docx)
            dir=$(dirname "$f")
            mkdir -p "docs/images/$filename"
            pandoc "$f" -s -o "$dir/$filename.md" -w gfm --extract-media="../images/$filename"
          done
```

This portion of the workflow is where the actual conversion happens. We recursively check each folder in the repository for files that end with the .docx extension. Then we get the base name of the file minus the file extension. **example.docx** ---> **example**. Then we save the directory of where we found the file. Next, we create a folder within the images folder to save all the extracted media that was found in the Microsoft Document. Note that the folder that holds all the images has the same name as the file and is not all just stored under images. This is because pandoc will not save the name of the image when it is extracted. It will label the image as **image1.png**, **image2.png**, etc. So extracting the media from several documents will cause the files to write over one another. That's why the images are stored in their respective folders.

```yml
      - name: Commit changes
        uses: stefanzweifel/git-auto-commit-action@v4
        with:
          commit_message: Changed Files
```

Commits the files to the repository.
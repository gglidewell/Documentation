# Automatic Conversion of Microsoft Documents to Markdown

## Pandoc 

The conversion of HTML Documents to Markdown is achieved using Pandoc, a command-line tool and library for converting documents from one format to another. Pandoc can extract the text from the document and create the relevant markdown to format the new file to mimic the document that it was converted. While these conversions are not always 100% accurate, they do mitigate the amount of work needed to transfer the information. This specifically deals with both HTML and HTM files, converting HTM files into HTML files before converting HTML to Markdown.

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
      - run: |
          find find -type f -name "*.htm" -print0 | while read -d $'\0' file; do
            iconv -f ISO-8859-1 -t UTF-8//TRANSLIT "$file" > "${file%%.htm}.html"     
             echo "Converted: $file to ${filename}.html"
          done
```


This is the conversion from HTM files into HTML files, Note that if an HTML file is uploaded this code does nothing.


```yml
      - run : |
          find find -type f -name "*.html" -print0 | while IFS= read -r -d '' f; do
            filename=$(basename "$f" .html)
            dir=$(dirname "$f")
            mkdir -p "docs/images/$filename"
            pandoc "$f" -s -o "$dir/$filename.md" -w gfm --extract-media="../images/$filename"
          done
```

This portion of the workflow is where the actual conversion happens. We recursively check each folder in the repository for files that end with the .html extension. Then we get the base name of the file minus the file extension. **example.html** ---> **example**. Then we save the directory of where we found the file. Next, we create a folder within the images folder to save all the extracted media that was found in the Microsoft Document. Note that the folder that holds all the images has the same name as the file and is not all just stored under images. This is because pandoc will not save the name of the image when it is extracted. It will label the image as **image1.png**, **image2.png**, etc. So extracting the media from several documents will cause the files to write over one another. That's why the images are stored in their respective folders.

```yml
      - name: Commit changes
        uses: stefanzweifel/git-auto-commit-action@v4
        with:
          commit_message: Changed Files
```

Commits the files to the repository.

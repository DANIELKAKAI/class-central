# class central mirrored site
This is a one level deep mirrored site of [classcentral](https://www.classcentral.com/) website which has been translated to Hindi language.

Tools used:
- wget (computer program that retrieves content from web servers)
- googletrans (Python package for translation)
- easynmt (Python package for translation)

## Prerequisites
- Download and install Python from the following website https://www.python.org/downloads/
- Download and install wget, check https://www.gnu.org/software/wget/

## Mirroring 
Use this wget commnand to download the website

`
wget -r -np -k -p --level=2 --user-agent="Mozilla" https://www.classcentral.com/
`

Let's break down this command:

- -r: Recursively download the website.

- -np: Do not ascend to the parent directory.

- -k: Convert the links in the HTML pages to local links.

- -p: Download all necessary files to display the page, such as images, stylesheets, and scripts.
- --level=2: Download one level deep

## Translation
Use either googletrans or easynmt packages for translation.
1. googletrans is fast but is error prone.
2. easynmt can work offline and is reliable but slow.

## Translation using googletrans
`
pip install googletrans==4.0.0rc1
`

Python code:
```python
from bs4 import BeautifulSoup
from googletrans import Translator
import os


def translate_to_hindi(file):
    print("Translating: ", file)
    # Load the HTML file
    with open(file, 'r') as f:
        html = f.read()

    # Parse the HTML using BeautifulSoup
    soup = BeautifulSoup(html, 'html.parser')

    # Find all the HTML tags containing the text to be translated
    tags = soup.find_all(string=True)

    # Translate the text within the tags from English to Hindi
    translator = Translator()
    for tag in tags:
        if tag.parent.name not in ['script', 'style']:
            # Exclude script and style tags, which should not be translated
            if tag.text in ('', '\n'):
                continue
            try:
                translated_text = translator.translate(tag, src='en', dest='hi').text
                tag.replace_with(translated_text)
            except IndexError:
                pass
    # Write the modified HTML back to the original file
    with open(file, 'w') as f:
        f.write(str(soup))


"""
Find all html files that need to be translated
"""

path = '{directory of mirrored site}'

files = []
for root, directories, filenames in os.walk(path):
    for filename in filenames:
        if '.html' in filename:
            files.append(os.path.join(root, filename))

for file in files:
    translate_to_hindi(file)

```

## Translation using easynmt
`
pip install easynmt
`

Python code:
```python
from bs4 import BeautifulSoup
import os
from easynmt import EasyNMT

model = EasyNMT('opus-mt')


def translate_to_hindi(file):
    print("Translating: ", file)
    # Load the HTML file
    with open(file, 'r') as f:
        html = f.read()

    # Parse the HTML using BeautifulSoup
    soup = BeautifulSoup(html, 'html.parser')

    # Find all the HTML tags containing the text to be translated
    tags = soup.find_all(string=True)

    # Translate the text within the tags from English to Hindi
    for tag in tags:
        if tag.parent.name not in ['script', 'style']:
            # Exclude script and style tags, which should not be translated
            if tag.text in ('', '\n'):
                continue
            translated_text = model.translate(tag, source_lang='en', target_lang='hi')
            tag.replace_with(translated_text)
    # Write the modified HTML back to the original file
    with open(file, 'w') as f:
        f.write(str(soup))


"""
Find all html files that need to be translated
"""

path = '{directory of mirrored site}'

files = []
for root, directories, filenames in os.walk(path):
    for filename in filenames:
        if '.html' in filename:
            files.append(os.path.join(root, filename))

for file in files:
    translate_to_hindi(file)

```

## Challenges
1. googletrans failures during translation.
2. easynmt slowness
3. The mirroring process is time consuming especially on low internet
4. The translation process is CPU intensive

In the end not all files were translated due to time.

## What could be improved
1. Implement multi-threading or multi-processing during translation for faster work
2. Use a more powerful CPU


# Nano Text Editor
---
When working from the command line, you’ll often need to create or modify text files.
Two of the most well-known and powerful command-line editors are Vim and Emacs, but both can be challenging to learn, especially for beginners. 
For users seeking a simpler option, there’s **nano**. GNU nano is a straightforward and user-friendly text editor designed for Unix and Linux systems. 
It offers essential features such as syntax highlighting, support for multiple file buffers, search and replace with regular expressions, spellcheck, UTF-8 encoding, and more.
This makes it an excellent choice for those who need a quick and easy editor without the complexity of more advanced tools.

## Opening and Creating Files 
To open an existing file or to create a new file, type nano followed by the file name:

                    nano filename

 This opens a new editor window, and you can start editing the file.

At the bottom of the window, there is a list of the most basic command shortcuts to use with the nano editor.

All commands are prefixed with either ^ or M characters. The caret symbol (^) represents the Ctrl key. For example, the ^J commands mean to press the Ctrl and J keys at the same time.
The letter M represents the Alt key.

You can get a list of all commands by typing Ctrl+g.

To open a file, you must have read permissions to the file.

## Editing Files
---
Unlike vi, nano is a modeless editor, which means that you can start typing and editing the text immediately after opening the file.

To move the cursor to a specific line and character number, use the Ctrl+_ command. The menu at the bottom of the screen will change. 
Enter the number(s) in the “Enter line number, column number:” field and hit Enter.

## Searching and replacing
---
To search for a text, press Ctrl+w, type in the search term, and press Enter. The cursor will move to the first match. To move to the next match, press Alt+w.

If you want to search and replace, press Ctrl+\. Enter the search term and the text to be replaced with. 
The editor will move to the first match and ask you whether to replace it. After hitting Y or N, it will move to the next match. Pressing A will replace all matches.

## Copping, cutting, and pasting
To select text, move the cursor to the beginning of the text and press Alt+a. This will set a selection mark. Move the cursor to the end of the text you want to select using the arrow keys. The selected text will be highlighted. If you wish to cancel the selection, press Ctrl+6.

Copy the selected text to the clipboard using the Alt+6 command. Ctrl+k will cut the selected text.


If you want to cut whole lines, move the cursor to the line and press Ctrl+k. You can cut multiple lines by hitting Ctrl+k several times.

To paste the text, move the cursor to where you want to put the text and press Ctrl+u.

## Saving and Exiting
To save the changes you’ve made to the file, press Ctrl+o. If the file doesn’t exist, it will be created once you save it.

To exit nano , press Ctrl+x. If there are unsaved changes, you’ll be asked whether you want to save the changes.

To save the file, you must have write permissions to the file. If you are creating a new file , you need to have write permission to the directory where the file is created.

                   

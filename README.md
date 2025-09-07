# MorseDecoder — Binary Decision Tree (C#)

A small C# console app that converts Morse code into text using a binary decision tree:

. means go left

- means go right

The character is stored at the node you reach

Example: … --- … (or ... --- ...) → SOS

1) What this project does

You paste a line of Morse code (dots and dashes).

The app walks a binary tree and prints the decoded letters and words.

It accepts normal spaces between letters and / (or 3+ spaces) between words.

It also cleans up pasted characters like … and — so they still work.

2) Why a binary decision tree?

Morse is binary: each step is either a dot or a dash.

Following the path . (left) and - (right) leads to the letter node.

This makes decoding fast and easy to understand.

3) What you need

.NET SDK 8+

(Optional) Visual Studio 2022 with “.NET desktop development”

4) Set up & run
Option A — Terminal
dotnet new console -n MorseDecoder
cd MorseDecoder
# Open Program.cs and paste in the provided code
dotnet run

Option B — Visual Studio

File → New → Project → Console App (.NET).

Name: MorseDecoder → Create.

Replace Program.cs content with the provided code.

Press F5 (Run).

5) Try these examples

SOS
Input: … --- … (or ... --- ...) → Output: SOS

HELLO WORLD
Input: .... . .-.. .-.. --- / .-- --- .-. .-.. -.. → Output: HELLO WORLD

ABC
Input: .- -... -.-. → Output: ABC

Rules

Single space = next letter

/ or 3+ spaces = next word

Unknown or invalid tokens print ?

6) Files you’ll see

Program.cs – everything in one file for simplicity

Node class: a tree node with .Dot and .Dash children

MorseTree class: builds the tree and decodes input

Main(): reads user input and prints the result

7) How it works (short version)

Build the tree using a map of A–Z, 0–9, and some punctuation.

For each letter’s Morse code (e.g., .- for A), insert it by walking the path:

. → create/go left

- → create/go right

To decode a token like .-:

Start at the root and follow the path

The final node stores the character

Combine characters into words using / or 3+ spaces.

8) Add your own characters

Open BuildDefault() and add a line:

tree.Insert('Ä', ".-.-"); // example


That’s it — the decoder will support it.

9) Common issues & fixes

dotnet not found: Install the .NET SDK and restart the terminal/PC.

Only ? shows: Check you used only ., -, spaces, and /. The app cleans … → ... and —/– → - automatically.

Word breaks wrong: Use / or 3+ spaces between words.

Weird paste characters: The app removes anything that isn’t ., -, /, or whitespace.

10) What to submit (typical assignment)

Your Program.cs

A screenshot of the app decoding at least 2 examples (e.g., SOS and HELLO WORLD)

A short paragraph (3–5 lines) explaining:

why a binary tree fits Morse

how . and - map to left/right

what ? means in your output

11) Stretch goals (optional)

Add encoding (text → Morse)

Add basic unit tests

Make a simple Windows Forms UI with a textbox and Decode button

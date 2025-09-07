# MorseDecoder — Binary Decision Tree (C#)

A clean, single-file **C# console app** that builds a **binary Morse decision tree** (dot = left, dash = right) and decodes Morse code into text. It also normalizes common Unicode variants like the ellipsis `…` and em/en dashes, so `… --- …` decodes to `SOS` out‑of‑the‑box.

---

## ✨ Features
- **Binary decision tree**: `.` → **left**, `-` → **right**; store symbols at terminal nodes.
- **Fast decode**: O(L) per letter (L ≤ ~5 for standard Morse).
- **Robust input**: Normalizes `…`, `—`, `–`, bullets, etc. to ASCII `.` and `-`.
- **Word breaks**: `/` or **3+ spaces** treated as word separators.
- **Fallbacks**: Unrecognized tokens become `?` so decoding never crashes.
- **Extensible**: A–Z, 0–9, and common punctuation included; easy to add more.

---

## 🧱 Project Structure
Single-file console app:
```
MorseDecoder/
└─ Program.cs
```

> If you build a Windows Forms front end, keep `Node` and `MorseTree` in a separate file (e.g., `MorseTree.cs`) and reuse them.

---

## 🚀 Getting Started

### Prerequisites
- **.NET SDK 8+**
- (Optional) **Visual Studio 2022** with “.NET desktop development”

### Create & Run (Terminal)
```bash
dotnet new console -n MorseDecoder
cd MorseDecoder

# Open Program.cs and paste in the code from this repo (see below).
dotnet run
```

### Create & Run (Visual Studio)
1. File → New → Project → **Console App (.NET)**.
2. Project name: `MorseDecoder` → Create.
3. Replace contents of `Program.cs` with the code in this repo.
4. Press **F5** (Run).

---

## 🖥️ Usage

When the app starts, paste a Morse sequence:

- **SOS**  
  Input: `… --- …` (or `... --- ...`)  
  Output: `SOS`

- **HELLO WORLD**  
  Input: `.... . .-.. .-.. --- / .-- --- .-. .-.. -..`  
  Output: `HELLO WORLD`

- **ABC**  
  Input: `.- -... -.-.`  
  Output: `ABC`

**Rules**  
- Letters are separated by **single spaces**.  
- Words are separated by **`/`** or **3+ spaces**.  
- Unknown tokens decode to **`?`**.

---

## 🧠 Design: Binary Decision (MiniMax) Tree
- The decoder uses a **binary decision tree**:
  - **`.`** = go **left** (`Dot` child)
  - **`-`** = go **right** (`Dash` child)
- The node you land on after following the full path stores the **symbol** (e.g., `'.-' → A`).
- Decoding a token walks the tree from the **root** following the token’s characters.

This structure yields predictable O(L) per token and avoids hashing or linear scans of the Morse table.

---

## 🧩 Implementation Notes

### Normalization
The app accepts pasted text using various Unicode characters and normalizes them to standard ASCII:
- `…` → `...`
- `—`, `–`, `−`, `_` → `-`
- `•`, `·`, `∙`, `●` → `.`
- It then **keeps only**: `.`, `-`, `/`, and whitespace.

### Word Boundaries
- `/` or **3+ spaces** produce a **space** in the output.
- Single spaces separate **letters**.

### Error Handling
- If a path doesn’t exist in the tree or a token contains invalid characters, the app returns **`?`** for that token.

---

## 🧪 Quick Check
Run the app and paste:
```
… --- …
```
Should print:
```
Decoded: SOS
```

---

## 🛠️ Adding Symbols
To add a new character, register it in `BuildDefault()`:
```csharp
tree.Insert('Ä', ".-.-"); // example
```
The decoder will immediately support it.

---

## 🪟 Optional: Windows Forms UI
You can wrap the same logic in a WinForms app:
1. Create **Windows Forms App (.NET 8)**.
2. Add UI controls: `TextBox` (Multiline, `txtMorse`), `Button` (`btnDecode`), `Label` (`lblOut`).
3. Move `Node` and `MorseTree` into `MorseTree.cs`.
4. In `Form1.cs`, on `btnDecode_Click`, call:
   ```csharp
   lblOut.Text = _tree.DecodeMessage(txtMorse.Text);
   ```

---

## 📦 Code (Program.cs)
> Paste this into your `Program.cs` (single-file app).

```csharp
using System;
using System.Collections.Generic;
using System.Text;
using System.Text.RegularExpressions;

namespace MorseDecoder
{
    class Program
    {
        static void Main(string[] args)
        {
            var morseTree = MorseTree.BuildDefault();

            Console.WriteLine("Morse Decoder");
            Console.WriteLine("Letters: separated by spaces. Use '/' or 3 spaces between words.");
            Console.WriteLine("Example: … --- …   (SOS)");
            Console.WriteLine();
            Console.Write("Enter Morse: ");

            string input = Console.ReadLine();
            if (string.IsNullOrWhiteSpace(input))
                input = "… --- …"; // demo

            string decoded = morseTree.DecodeMessage(input);
            Console.WriteLine($"Decoded: {decoded}");
            Console.WriteLine();
            Console.WriteLine("Press ENTER to exit.");
            Console.ReadLine();
        }
    }

    // Binary decision node: '.' -> Dot (left), '-' -> Dash (right)
    class Node
    {
        public char? Symbol;
        public Node Dot;
        public Node Dash;
    }

    class MorseTree
    {
        private readonly Node _root = new Node();
        private MorseTree() {}

        public void Insert(char symbol, string code)
        {
            var cur = _root;
            foreach (char c in code)
            {
                switch (c)
                {
                    case '.':
                        cur.Dot ??= new Node();
                        cur = cur.Dot;
                        break;
                    case '-':
                        cur.Dash ??= new Node();
                        cur = cur.Dash;
                        break;
                    default:
                        throw new ArgumentException($"Invalid Morse character '{c}' in code for {symbol}.");
                }
            }
            cur.Symbol = symbol;
        }

        public char DecodeToken(string token)
        {
            var cur = _root;
            foreach (char raw in token)
            {
                char c = raw switch { '.' => '.', '-' => '-', _ => '?' };

                if (c == '.') cur = cur.Dot;
                else if (c == '-') cur = cur.Dash;
                else return '?';

                if (cur == null) return '?';
            }
            return cur.Symbol ?? '?';
        }

        public string DecodeMessage(string input)
        {
            string normalized = NormalizeInput(input);

            // 3+ spaces => word separator
            normalized = Regex.Replace(normalized, "\\s{3,}", " / ");
            // collapse multiple spaces
            normalized = Regex.Replace(normalized, "\\s+", " ").Trim();

            var sb = new StringBuilder();
            foreach (var token in normalized.Split(' '))
            {
                if (token == "/") { sb.Append(' '); continue; }
                if (token.Length == 0) continue;

                sb.Append(DecodeToken(token));
            }
            return sb.ToString();
        }

        private static string NormalizeInput(string s)
        {
            if (string.IsNullOrEmpty(s)) return s;

            // Normalize unicode variants to ASCII '.' and '-'
            s = s.Replace("…", "...")
                 .Replace("•", ".").Replace("·", ".").Replace("∙", ".").Replace("●", ".")
                 .Replace("—", "-").Replace("–", "-").Replace("−", "-").Replace("―", "-").Replace("_", "-");

            // Keep only dot, dash, slash, and whitespace
            var cleaned = new StringBuilder(s.Length);
            foreach (char c in s)
                if (c == '.' || c == '-' || c == '/' || char.IsWhiteSpace(c))
                    cleaned.Append(c);

            return cleaned.ToString();
        }

        public static MorseTree BuildDefault()
        {
            var tree = new MorseTree();

            var map = new Dictionary<char, string>
            {
                ['A'] = ".-",    ['B'] = "-...",  ['C'] = "-.-.",  ['D'] = "-..",
                ['E'] = ".",     ['F'] = "..-.",  ['G'] = "--.",   ['H'] = "....",
                ['I'] = "..",    ['J'] = ".---",  ['K'] = "-.-",   ['L'] = ".-..",
                ['M'] = "--",    ['N'] = "-.",    ['O'] = "---",   ['P'] = ".--.",
                ['Q'] = "--.-",  ['R'] = ".-.",   ['S'] = "...",   ['T'] = "-",
                ['U'] = "..-",   ['V'] = "...-",  ['W'] = ".--",   ['X'] = "-..-",
                ['Y'] = "-.--",  ['Z'] = "--..",
                ['0'] = "-----", ['1'] = ".----", ['2'] = "..---", ['3'] = "...--",
                ['4'] = "....-", ['5'] = ".....", ['6'] = "-....", ['7'] = "--...",
                ['8'] = "---..", ['9'] = "----."
            };

            var punctuation = new Dictionary<char, string>
            {
                ['.'] = ".-.-.-", [','] = "--..--", ['?'] = "..--..", ['!'] = "-.-.--",
                ['/'] = "-..-.",  ['('] = "-.--.",  [')'] = "-.--.-", ['&'] = ".-...",
                [':'] = "---...", [';'] = "-.-.-.", ['='] = "-...-",  ['+'] = ".-.-.",
                ['-'] = "-....-", ['_'] = "..--.-", ['\"'] = ".-..-.", ['$'] = "...-..-",
                ['@'] = ".--.-."
            };

            foreach (var kv in map) tree.Insert(kv.Key, kv.Value);
            foreach (var kv in punctuation) tree.Insert(kv.Key, kv.Value);

            return tree;
        }
    }
}
```

---

## 📚 Theory/Notes
- **Time complexity**: O(N·L) to decode a message with N tokens and average code length L.
- **Space complexity**: O(S) for number of unique symbols supported (each creates nodes along its path).

---

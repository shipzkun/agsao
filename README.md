# Agsao
All-JS custom passphrase generator, inspired by Bitwarden's passphrase generator.

As this runs purely client-side (i.e., in browser), it won't log the generated passphrases in a remote server. The only remote request it does is when loading wordlists; after that, no HTTP requests will be performed.

The library is coded in vanilla JS (i.e., no jQuery, lodash, or other JS frameworks). 

This is intended to run inside modern browsers. The following modern JS features are used:
- async/await
- Promises
- for...in
- for...of 
- JS classes
- JS private class features
- anonymous functions

I personally tested this in latest versions of Firefox and MS Edge. Feel free to test in other browsers. This might also help: https://caniuse.com/.

It might run in nodeJS or other non-browser contexts. Since I had no need for those, I haven't tested this in those contexts. Again, feel free to test.


Live usage: https://arden-chan.party/ilo_agsao


# Quick Setup
```html
<!-- Import the module. type="module" is important! -->
<script src="/path/to/agsao.js" type="module"></script>
<!-- You will also need an entropy calculator. Import it now. -->

<!-- 
General flow:
- Load the library
- Set the configs (explained in sections below):
  - entropy_calculator: function to compute generated passphrase entropy
  - configs: library configuration controlling how passphrases are generated
  - wordlists: array of URLs to list of words
- Generate phrases, as many as you want.
-->

<script>
Agsao a = new Agsao()
  
a.entropy_calculator = entropy_calculator_function 
a.configs = {        // library defaults
  wordlen_min : 4,
  wordlen_max : 7,
  addnum : "end",
  addspecial : "no",
  allow_dupe_words : true,
  capitalise_words : true,
  num_words : 3,
  separator : "|",
  phraselen_min : 12,
  phraselen_max : 70,
  min_entropy_bits : 80
}
a.wordlists = ["http://wordlist1_url", "http://wordlist2_url"]

// For example, if we want to generate 10 passphrases using the settings above:
for (let i=0; i<10; i++) {
  // NOTE: generate() is async
  a.generate().then(res => {
    if (!res.success) {
      console.log("Error generating phrase: "+res.msg)
    } else {
      console.log("Passphrase: "+res.data.phrase)
      console.log("Entropy: "+res.data.entropy)
    }
  });   
}
</script>
```

# Configurations
```js
a.entropy_calculator = entropy_calculator_function 
a.configs = {        // library defaults
  wordlen_min : 4,
  wordlen_max : 7,
  addnum : "end",
  addspecial : "no",
  allow_dupe_words : true,
  capitalise_words : true,
  num_words : 3,
  separator : "|",
  phraselen_min : 12,
  phraselen_max : 70,
  min_entropy_bits : 80
}
a.wordlists = ["http://wordlist1_url", "http://wordlist2_url"]
```
- `entropy_calculator`: a JS function to calculate passhprase entropy. 
  - This must be a function that accepts a passphrase and outputs an entropy number. 
  - For instance, using https://github.com/EYHN/PasswordQualityCalculator, you can set:
    ```js
    // assuming you've imported the library above using
    // import * as kp_pqc from "/path/to/PasswordQualityCalculator.js";
    a.entropy_calculator = kp_pqc.default
    ```
- `wordlists`: an array of URLs containing the wordlists
  - The library expects one word per line.
  - Since wordlists are retrieved by the browser, CORS might be relevant.
- `configs`
  - `wordlen_min`: minimum number of characters a given word should have
  - `wordlen_max`: maximum number of characters a given word should have
  - `addnum`: whether or not to add a number in a word, and where. Possible values: `[null, "start", "end", "random"]`
    - This does not guarantee a word will have a number in it. If it does, then this controls where that randomly-generated number is placed.
    - Only one digit from 0-9 is added to the word.
  - `addspecial`: whether or not to add a special character in a word, and where. Possible values: `[null, "start", "end", "random"]`
    - This does not guarantee a word will have a special character in it. If it does, then this controls where that randomly-generated special character is placed.
    - Only one character from `~!@#$%^&*()_+`-='\"{}[]\\|<>,./?` is added to the word.
  - `allow_dupe_words`: Allow a word to be duplicated in a passphrase. `true` if yes, `false` if no.
  - `capitalise_words`: Whether or not to capitalise the word. `true` if yes, `false` if no. This uses your browser's locale for capitalisation rules (you might want to test if this fits your needs).
  - `num_words`: The number of words to include in the passphrase.
  - `separator`: Separator character(s) between words. This is not limited to a single character; you may put whatever you like.
  - `phraselen_min`: Minimum passphrase length.
  - `phraselen_max`: Maximum passphrase length. If the original passphrase is longer than this, the extra characters are dropped in the final phrase.
  - `min_entropy_bits`: The minimum number of entropy bits the passphrase should have.

# Methods
- `validate_configs()`: validates if the set configurations above are correct. If everything is OK, it returns nothing; otherwise, it will return an Object of the format `{"key": "error message for the key"}`. Possible keys:
  - `wordlists`
  - `wordlen_min`
  - `wordlen_max`
  - `addnum`
  - `addspecial`
  - `num_words`
  - `phraselen_min`
  - `phraselen_max`
  - `min_entropy_bits`
- `async generate()`: Generates the actual passphrase. Returns an Object of the format `{"success": true|false, "msg": "error message", "data":{"phrase":"generated passphrase", "entropy": "passphrase_entropy_int"}}`.
  - `success`: whether or not the generation succeeded. `true` if yes, `false` if not.
  - `msg`: error message why the generation failed
  - `data`
    - `phrase`: The generated passphrase
    - `entropy`: The entropy of the generated passphrase 
  - This function is asynchronous and returns a Promise. You will need to handle that accordingly, like so:
  ```js
  a.generate().then(res => {
    if (!res.success) {
      console.log("Error generating phrase: "+res.msg)
    } else {
      console.log("Passphrase: "+res.data.phrase)
      console.log("Entropy: "+res.data.entropy)
    }
  });   
  ```
  
# What's with the name?
*Agsao* <code>[ɐg.sɐ.'o]</code> means *to speak* in Ilocano. Given the nature of the service, it's close enough, don't you think? 

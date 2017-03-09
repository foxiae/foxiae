---
layout: single

categories: security

title: "Understanding Password Complexity"

excerpt: "Understand what makes strong passwords and protect your online presence."
---
What does it mean to have a "strong" password? How can you be sure your password can't be compromised? How do websites know when your password is complex enough?

These are questions I started asking myself in the beginning in February. And while it may sound trivial, I wanted to understand *how* those strength bars know how good my password is.

So let's start with entropy. Entropy, in the Information Theory sense, measures how unpredictable text is based on the character set used and its length. And to answer these questions, we'll need to look at password entropy, which measures the *strength* of the password generation. Meaning, the results from our calculations will signify how hard it is to guess a password under certain assumptions. This will all make sense when we take a look at how password entropy is calculated.

## Let's Talk About Math
Now let's build out our function. We'll need two variables - one for the length of our password (denoted as *L*) and another for the set of possible combinations with our dictionary (denoted as *N*). Dictionary, in this sense, means the character set used to generate the password, such as numbers (0-9) to create a PIN or your typical ASCII printable characters (a-z, A-Z, 0-9, and special characters).

So, to calculate the number of possible passwords we'll need to raise *N* by the power of *L*, or *N<sup>L</sup>*. The is the foundation of our equation. (*N.B.:* If we increase *L* or *N*, we'll increase the number of possible passwords, which will thereby increase the strength of our passwords generated. This will be demonstrated a bit later.)

Assuming that each symbol within the password is produced independently, we'll use the [binary logarithm](https://en.wikipedia.org/wiki/Binary_logarithm), as the password (in the end) is in *bits*, on our foundation to determine the entropy. So:

$$
{entropy\,\,of\,\,password}\, =\, \log_{ 2 } N^L
$$

With a little bit of manupilations, we can get the following result:

$$
{entropy\,\,of\,\,password}\,  =\, \log_{ 2 } N^L =\, L\;(\log_{ 2 } N)\, = \, L\; \frac{ \log_{} N }{ \log_{} 2 }
$$

Taking a step back, our equation resembles a rough version of [Shannon's entropy](https://en.wiktionary.org/wiki/Shannon_entropy), which determines the probability of characters being represented within the password based on the sum of each individual character's entropy. In short, Shannon's entropy has guided our evolution for our password generation entropy equation.

### How Easily Can Your Password be Guessed?
Well, it depends on the dictionary your password uses. So let's put our new found powers to use!

Let's take for example your normal password requirements of 8 - 16 characters, requiring at least one capital letter, a number, and a special character. What's the *max* entropy we can generate? Time to churn our equation. Our dictionary set is 95 characters, and we'll assume the max values for the password length and that the characters are independently generated from each other:

$$
16\; \frac{ \log_{} 95 }{ \log_{} 2 } = 105
$$

Hence, the entropy of this password is 105 bits. Well, that's all fine and well, but *what does this result mean?* I'm glad you thought that.

To calculate the results, we'll use the following equation (you'll notice it's the inverse of our password entropy equation):

$$
seconds\,\,to\,\,guaranteed\,\,crack\,\,=\,\,\frac{ 2^{\;entropy} }{ guesses\,\,per\,\,second }
$$

In 2012, a password-cracking [expert unveiled a computer](https://arstechnica.com/security/2012/12/25-gpu-cluster-cracks-every-standard-windows-password-in-6-hours/) that could cycle through 350 billion guesses per second. So, as an extreme case, let's use this number:

$$
\frac{ 2^{105} }{ 350,000,000,000 } = \frac{ 9614081257132168796771975168 }{ 341796875 }\;seconds \,\, \approx \,\, 891.9\,\,billion\,\,years
$$

Geez! 105 password entropy seems relatively safe. Well, this is all on the assumption that this attack is happening offline (as *most* server-side securities prevents multi-failed attempts of password guessing) or cracking something locally encrypted, and this is assuming there isn't a stronger password cracker out there now. Notice that the entropy is only stronger when the `guesses per second` is weaker.

So how much entropy is recommended for everyday use? Well, that depends on the application and its use. There seems to be a common census to use 80 bits of entropy for everyday use. Again, the stronger the better here. It should also go without saying that [weak passwords](https://github.com/danielmiessler/SecLists/tree/master/Passwords) should be excluded from your password generation.

### How Can I Get a Really High Entropy Password?
Use a big dictionary. Remember that the more combinations your dictionary has, the stronger your potential entropy.

One of the biggest dictionaries with 7776 short words is [Diceware](http://world.std.com/~reinhold/diceware.html). With this method, you could calculate a Diceware password yourself with five rolls of a dice. The end result of all these rolls will correspond to a word from Diceware's dictionary (e.g., a `16656` roll corresponds to `claw`). These words are short so that you can easily remember them. As the [XKCD](https://xkcd.com/936/) comic eloquently put it:

![](https://imgs.xkcd.com/comics/password_strength.png)

Let's take a moment though to create a table for the needed amount of characters to achieve a certain amount of entropy[^1]:

**Desired Entropy**|**Numbers (0-9)**|**Hexadecimal (0-9, A-F)**|**Case insensitive Latin alphabet (a-z or A-Z)**|**Case insensitive alphanumeric (a-z or A-Z, 0-9)**|**Case sensitive Latin alphabet (a-z, A-Z)**|**Case sensitive alphanumeric (a-z, A-Z, 0-0)**|**All ASCII printable characters (without space)**|**All extended ASCII printable characters**|**Diceware word list**
:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:
8 bits |3|2|2|2|2|2|2|2|1
32 bits |10|8|7|7|6|6|5|5|3
40 bits |13|10|9|8|8|7|7|6|4
64 bits |20|16|14|13|12|11|10|9|5
80 bits |25|20|18|16|15|14|13|11|7
96 bits |29|24|21|19|17|17|15|13|8
128 bits |39|32|28|25|23|22|20|17|10
160 bits |49|40|35|31|29|27|25|21|13
192 bits |58|48|41|38|34|33|30|25|15
224 bits |68|56|48|44|40|38|35|29|18
256 bits |78|64|55|50|45|43|39|33|20

Clearly, the winner here with the fewest amount of characters needed is Diceware, but coming in second is ASCII characters as one would expect.

As measuring entropy relies on a randomly generated string. People are notoriously bad at generating random passwords. Our "randomness" will most likely come from things we use most often - like vowels. [One analysis](http://ijmcs.info/current_issue/IJMCS140807.pdf) showed that over 3 million eight-character passwords, the letter "e" was used over 1.5 million times, where the letter "f" was only used 250,000 times. Ideally, if characters were evenly distributed throughout these passwords, each character would only be used around 900,000 times.

In short? Store your randomly generated password with [password manager](https://en.wikipedia.org/wiki/List_of_password_managers), create your passwords with a [random password generator](https://en.wikipedia.org/wiki/Random_password_generator), and don't use commonly known passwords.

[^1]:
      The table can be found https://en.wikipedia.org/wiki/Password_strength#Random_passwords

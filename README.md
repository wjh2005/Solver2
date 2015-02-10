Solver2
===================

This readme is a tribute to [Letters](https://itunes.apple.com/us/app/letters-game-about-spelling/id823334911?ls=1&mt=8) and describes how I came to achieve the global high score in this game.

Letters is an iPhone game by [Chad](https://twitter.com/jazzychad) [Etzel](http://jazzychad.net). The game presents a grid of twenty-five randomly selected letters to the player, who taps these letters to spell as many twelve-letters-or-fewer English-language words as possible in the time allotted per round, initially sixty seconds. Each word spelled increases the player's score. The score of each word is based on the point values of its letters, which range from one to ten, and multiplier letters, of which there are four. A light blue letter is worth triple the normal letter value. A light red letter is worth double the normal letter value. A dark red letter doubles the value of the word. A dark blue letter triples the value of the word. The player can invoke three power-ups during gameplay. Quoting the in-app documentation: "Clock Extenders add 10 seconds to the clock so [the player has] time to spell more words." "Letter Destroyers let [the player] remove a letter from the board so [the player] can put high-scoring letters on bonus tiles." "Extra Bonus Tiles will place an additional bonus tile on the board to score huge points for each word." The player can purchase power-ups using either points accrued in past games or real money via in-app purchase.

Having heard about Letters on [iOhYes](http://iohyespodcast.com), a software-development podcast Mr. Etzel co-hosts, I gave Letters a whirl. Letters became my go-to iPhone game for a time. I enjoyed the excitement of racing the [clock](http://watches.uhrzeit.org/atomic-clock.php) to spell one last word at the end of every round. The bouncy animation effects hypnotized me. I appreciated the tasteful business model. (The player need not make in-app purchases to fully experience Letters, as in [Crossy Road](https://itunes.apple.com/app/crossy-road-endless-arcade/id924373886?at=1l3vs3u&ct=crcom).)

As a goal-oriented person, I dreamed of achieving the global high score for Letters. At the time I began playing, the global high score, by a gent named [Michael Creasy](https://twitter.com/crsy), was 4232. My unassisted high score topped out around 1000, and my average was around 500. I clearly had some work to do. In one (presumably) seventy-second [round](https://twitter.com/crsy/status/466625449724891136), Mr. Creasy was able to spell the words punt, zee, wheel, pin, oily, thine, rods, jazzy, vine, mu, book, web, hi, re, ray, better, zone, ten, pow, cries, ion, dot, cap, quint, op, tear, and hazed.

I had an idea. What if I were to screenshot the Letters board, go back to the Home screen, thereby pausing the game, use the Internet to figure out the best word to spell, and input that word in Letters? I found a website, [WordFind](http://www.wordfind.com), that takes as input twelve letters and outputs words that can be spelled with those letters.

![alt text](https://github.com/vermont42/Solver2/tree/master/images/WordFind.png “WordFind”)

Using words suggested by WordFind, I was able to consistently score 1000 points per game and sometimes as many as 1500. This technique had problems, though. First, WordFind only accepts twelve letters, not the twenty-five that Letters provides. By giving WordFind just twelve letters, I was arbitrarily constraining the universe of possible words. (How did I choose those twelve letters? Guesswork. I started with the power-up letters and filled in the remaining seven or eight letters with a mix I thought likely to spell English words and result in high scores.) Second, I had no way of knowing how much each of the words output by WordFind was worth in Letters because the website was unaware of Letters' letter values or power-ups. I therefore had to guess using the presence of power-up and high-value letters in candidate words.

I decided that software could solve the second problem. I created an app, Letters Solver 1 (LS1), that took as input the power-ups on the board and a candidate word and then output the word's score.

![alt text](https://github.com/vermont42/Solver2/tree/master/images/LS1.png “Letters Solver 1”)

LS1 removed the guesswork as to word score but did not address the twelve-letter limit inherent in WordFind.

I did some digging on the App Store and found [WordsFinder](http://janswaal.home.xs4all.nl/iPhone/WordsFinder/). This powerful app does many things, but the feature of interest to me was that I could input all twenty-five letters on the Letters board and get _all_ ten-to-twelve-letter English-language words that could be spelled with them. I plugged candidate words from WordsFinder into LS1 and got my high score up to about 2000. Although this technique did increase my high score, there were two problems with it. First, the process of inputting letters on the board into WordsFinder and then plugging candidate words into LS1 was extremely tedious. Second, WordsFinder, by limiting its output to words of at least ten letters in length, was artificially limiting the average score per letter. Consider the following example: The letters on the board are uvoeiendbttiidqsooawmpcma. Q is light blue, u is light red, i is dark red, and t is dark blue. According to WordsFinder and LS1, "antimosquito" may be the most valuable twelve-letter word; its total score is 288, with an average score per letter of 24. But "quit" is worth _54_ points per letter, and it's hecka easier to input in Letters. (I came to find the process of inputting suggested twelve-letter words in Letters stressful and error-prone because of the time constraint.) Thus, "quit" is arguably a better choice than "antimosquito" because it gives more bang for the buck and is less likely to result in a mis-tap when inputting the word in Letters. But WordsFinder doesn't show four-letter words and is unaware of Letters point values or power-ups and therefore does not suggest the "better" word.

![alt text](https://github.com/vermont42/Solver2/tree/master/images/WordsFinder.png “WordsFinder”)

LS1, working in concert with WordFind and WordsFinder, had helped me achieve some good scores, but I realized that if I was going to beat Mr. Creasy’s high score, I needed to completely roll my own solution.

As I envisioned it, Letters Solver 2 (LS2), the sequel to LS1, would take as input a PNG screenshot of Letters, figure out out what letters and what color letters were in the screenshot by comparing twenty-five regions of the screenshot with 130 (26 letters x 5 colors) pre-captured letter PNGs, and output the best (highest-score-per-letter) word. Because I had never written any code involving image processing, this part was going to be challenging to implement, but I welcomed this departure from my programming comfort zone.

I started out by looking for third-party code to find PNGs within PNGs. I found an open-source library called OpenCV that might have done the job, but I couldn't get it to build, notwithstanding my ability to find ]the OpenCV build error in a Google search. Not knowing if OpenCV's performance was going to be acceptable for LS2, I was unwilling to invest more than a couple hours fixing the build error, at least not initially.

I found some C# [code](http://www.codeproject.com/Articles/38619/Finding-a-Bitmap-contained-inside-another-Bitmap) for finding PNGs within PNGs. Because C# is similar to ([some](http://news.cnet.com/2100-1082-817522.html) would say a debased copy of) Java, a language I already knew, I decided that adapting this code for LS2 wouldn't be terribly difficult.

I considered comparing the bits in the screenshot to bits in the pre-captured letter PNGs. The Wikipedia PNG article had a link to the PNG [spec](http://www.w3.org/TR/PNG/). It turned out that PNG uses a gnarly compression algorithm, so this wasn't going to be easy.

Finally I had an epiphany: how the bits are represented in a PNG in long-term storage is completely different from how the bits of a UIImage are represented in a running app. I found on StackOverflow some code for comparing the bits of UIImages, and I decided to appropriate it and use the approach of having the runtime convert the screenshot and letter PNGs to UImages, crop twenty-five UIImages from each screenshot, and compare the cropped UIImages to the letter PNGs using the StackOverflow code.

The first task was to figure out the coordinates and size of the twenty-five letters within each screenshot. Using Gimp I figured out the locations of the letters and that they were 50x50 pixels. I then took approximately 500 screenshots of Letters and began the process of pre-capturing the 130 letter images I needed. Why so many screenshots? 200 probably would have been enough. The answer is testing. Cropping the letter images from the screenshots was a potentially error-prone process. I wanted a way to find out if I had cropped a letter incorrectly. So I decided to get a screenshot of at least two different rounds with each of the 130 letter images: one screenshot as the source of each letter, the other screenshot to verify that I had correctly cropped. As I cropped letters, therefore, I used the StackOverflow code to verify, using the second screenshot, that I had cropped the correct region. This approach bore fruit, as it turned out on two occasions that I had not.

The last letter I cropped was a light-blue q. With that image in hand, I adapted some code that Dave DeLong had put on StackOverflow to read a list of 239,022 valid Scrabble words from long-term storage and put them in an NSArray. I then adapted the LS1 code to check whether each of the 239,022 words was spellable and, if so, its value per letter, reporting the best word back to the user.

I realized that the LS2 user might want to constrain the maximum length of the best word if the user were about to run out of time in a Letters round. I also realized that the user might want to constrain the minimum length in order to reduce the time cost of switching between Letters and LS2. I therefore made those two constraints settable in the LS2 UI.

Initially, I put the maximum and minimum lengths in editable UITextFields, but this approach left me unsatisfied. I wanted the tactile, error-proof number-setting experience that UISlider would provide if it supported setting a range of values. This desire led me down a fruitful rabbit hole: I ended up implementing an open-source range slider that got featured on [link to ManiacDev and formal name]. I considered using one of the open-source range sliders that already existed, but I wanted to learn how to create an [IBInspectable/IBDesignable](http://nshipster.com/ibinspectable-ibdesignable/) control, and I did.

With LS2 complete, I started attacking Mr. Creasy’s high score. As this [video](https://vimeo.com/115927968) attests, the process of using LS2 is both painless and jazzy. On my eleventh attempt, I achieved my goal by receiving a score of 6696, 2464 points higher than Mr. Creasy’s best. This Game Center screenshot is the proof: [screenshot] The avatar you see is my Tonkinese cat, Sandy. In this round, LS2 suggested vampirize, plaque, taxable, quiz, pokeful, subitize, qorma, sucking, pinecone, puja, evzone, and jeton. I consider my vocabulary pretty good, but I have never heard or seen six of those words, and I would therefore not have thought of them a possible Letters words, even without time pressure.

![alt text](https://github.com/vermont42/Solver2/tree/master/images/GameCenter.png “Game Center”)

Notwithstanding this success, there are ways I could improve LS2. For example, I could have it grab the latest screenshot from the camera roll automatically rather than having the user manually select it. I could use a route-planning algorithm to suggest to the user the shortest (in the spatial sense) sequence of letters to tap for any word suggested. I could build a robot to input the suggested word more quickly than any human could do.

Leonardo da Vinci once said that no work of art is ever complete, only abandoned. I do not consider LS2 a work of art, but I do not plan to implement new features for it. Rather, I plan to take what I learned from creating LS2 to build my next app, one with a less-nefarious purpose and a broader appeal, perhaps one for learning Spanish verb conjugations or tracking competitors in running races.

Not-so-brief aside: In a typical run, LS2 stores the RGB values of 387,500 pixels and performs 158,428 comparisons against these values. In my initial implementation, I stored the RGB values as 1,162,500 NSNumbers in two NSMutableArrays. A typical run was taking twenty seconds, longer than I wanted. So I tried storing the values as 1,162,500 floats in two C arrays. Run time dropped to two seconds. This massive performance boost causes me to have the following concern about using Swift for my next app. As I understand it, Swift does not allow the use of C arrays. If I had written LS2 in Swift, performance might have been slow unless I had put the pixel-munging code in a separate, Objective-C file, something I would prefer not to do.


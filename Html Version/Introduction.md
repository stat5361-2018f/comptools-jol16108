\mainmatter



# Introduction

With more than 10 years experience programming in R, I've had the luxury of being able to spend a lot of time trying to figure out and understand how the language works. This book is my attempt to pass on what I've learned so that you can quickly become an effective R programmer. Reading it will help you avoid the mistakes I've made and dead ends I've gone down, and will teach you useful tools, techniques, and idioms that can help you to attack many types of problems. In the process, I hope to show that, despite its frustrating quirks, R is, at its heart, an elegant and beautiful language, well tailored for data analysis and statistics.

If you are new to R, you might wonder what makes learning such a quirky language worthwhile. To me, some of the best features are:

* It's free, open source, and available on every major platform. As a result, if 
  you do your analysis in R, anyone can easily replicate it.

* A massive set of packages for statistical modelling, machine learning,
  visualisation, and importing and manipulating data. Whatever model or
  graphic you're trying to do, chances are that someone has already tried
  to do it. At a minimum, you can learn from their efforts.

* Cutting edge tools. Researchers in statistics and machine learning will often
  publish an R package to accompany their articles. This means immediate
  access to the very latest statistical techniques and implementations.

* Deep-seated language support for data analysis. This includes features
  like missing values, data frames, and subsetting.

* A fantastic community. It is easy to get help from experts on the
  [R-help mailing list](https://stat.ethz.ch/mailman/listinfo/r-help),
  [stackoverflow](http://stackoverflow.com/questions/tagged/r), [RStudio Community](https://community.rstudio.com/),
  or subject-specific mailing lists like
  [R-SIG-mixed-models](https://stat.ethz.ch/mailman/listinfo/r-sig-mixed-models)
  or [ggplot2](https://groups.google.com/forum/#!forum/ggplot2). You
  can also connect with other R learners via
  [twitter](https://twitter.com/search?q=%23rstats),
  [linkedin](http://www.linkedin.com/groups/R-Project-Statistical-Computing-77616),
  and through many local
  [user groups](https://jumpingrivers.github.io/meetingsR/).

* Powerful tools for communicating your results. R packages make it easy to
  produce html or pdf [reports](http://yihui.name/knitr/), or create
  [interactive websites](http://www.rstudio.com/shiny/).

* A strong foundation in functional programming. The ideas of functional
  programming are well suited to solving many of the challenges of data
  analysis. R provides a powerful and flexible toolkit which allows
  you to write concise yet descriptive code.

* An [IDE](http://www.rstudio.com/ide/) tailored to the needs of interactive
  data analysis and statistical programming.

* Powerful metaprogramming facilities. R is not just a programming language, it
  is also an environment for interactive data analysis. Its metaprogramming
  capabilities allow you to write magically succinct and concise functions and
  provide an excellent environment for designing domain-specific languages.

* Designed to connect to high-performance programming languages like C,
  Fortran, and C++.

Of course, R is not perfect. R's biggest challenge is that most R users are not programmers. This means that:

* Much of the R code you'll see in the wild is written in haste to solve
  a pressing problem. As a result, code is not very elegant, fast, or easy to
  understand. Most users do not revise their code to address these shortcomings.

* Compared to other programming languages, the R community tends to be more
  focussed on results instead of processes. Knowledge of software engineering
  best practices is patchy: for instance, not enough R programmers use source
  code control or automated testing.

* Metaprogramming is a double-edged sword. Too many R functions use
  tricks to reduce the amount of typing at the cost of making code that
  is hard to understand and that can fail in unexpected ways.

* Inconsistency is rife across contributed packages, even within base R.
  You are confronted with over 20 years of evolution every time you use R. 
  Learning R can be tough because there are many special cases to remember.

* R is not a particularly fast programming language, and poorly written R code
  can be terribly slow. R is also a profligate user of memory. 

Personally, I think these challenges create a great opportunity for experienced programmers to have a profound positive impact on R and the R community. R users do care about writing high quality code, particularly for reproducible research, but they don't yet have the skills to do so. I hope this book will not only help more R users to become R programmers but also encourage programmers from other languages to contribute to R.

## Who should read this book {#who-should-read}

This book is aimed at two complementary audiences:

* Intermediate R programmers who want to dive deeper into R and learn new
  strategies for solving diverse problems.

* Programmers from other languages who are learning R and want to understand
  why R works the way it does.

To get the most out of this book, you'll need to have written a decent amount of code in R or another programming language. You might not know all the details, but you should be familiar with how functions work in R and although you may currently struggle to use them effectively, you should be familiar with the apply family (like `apply()` and `lapply()`).

This books works the narrow line between being a reference book (primarily used for lookup), and being linearly readable. This involves some tradeoffs, because it's difficult to linearise material while still keeping related materials together, whereas some concepts are much easier to explain if you're already familiar with specific technically vocabulary. I've tried to use footnotes and cross-references to make sure you can still make sense even if you just dip your toes in the occassional chapter.  

## Related work

Tidyverse + R4DS

R packages

## What you will get out of this book {#what-you-will-get}

This book describes the skills I think an advanced R programmer should have: the ability to produce quality code that can be used in a wide variety of circumstances.

After reading this book, you will:

* Be familiar with the fundamentals of R. You will understand complex data types
  and the best ways to perform operations on them. You will have a deep
  understanding of how functions work, and be able to recognise and use the four
  object systems in R.

* Understand what functional programming means, and why it is a useful tool for
  data analysis. You'll be able to quickly learn how to use existing tools, and
  have the knowledge to create your own functional tools when needed.

* Appreciate the double-edged sword of metaprogramming. You'll be able to
  create functions that use non-standard evaluation in a principled way,
  saving typing and creating elegant code to express important operations.
  You'll also understand the dangers of metaprogramming and why you should be
  careful about its use.

* Have a good intuition for which operations in R are slow or use a lot of
  memory. You'll know how to use profiling to pinpoint performance
  bottlenecks, and you'll know enough C++ to convert slow R functions to
  fast C++ equivalents.

* Be comfortable reading and understanding the majority of R code.
  You'll recognise common idioms (even if you wouldn't use them yourself)
  and be able to critique others' code.

## Meta-techniques {#meta-techniques}

There are two meta-techniques that are tremendously helpful for improving your skills as an R programmer: reading source code and adopting a scientific mindset.

Reading source code is important because it will help you write better code. A great place to start developing this skill is to look at the source code of the functions and packages you use most often. You'll find things that are worth emulating in your own code and you'll develop a sense of taste for what makes good R code. You will also see things that you don't like, either because its virtues are not obvious or it offends your sensibilities. Such code is nonetheless valuable, because it helps make concrete your opinions on good and bad code.

A scientific mindset is extremely helpful when learning R. If you don't understand how something works, develop a hypothesis, design some experiments, run them, and record the results. This exercise is extremely useful since if you can't figure something out and need to get help, you can easily show others what you tried. Also, when you learn the right answer, you'll be mentally prepared to update your world view. When I clearly describe a problem to someone else (the art of creating a [reproducible example](http://stackoverflow.com/questions/5963269)), I often figure out the solution myself.

## Recommended reading {#recommended-reading}

R is still a relatively young language, and the resources to help you understand it are still maturing. In my personal journey to understand R, I've found it particularly helpful to use resources from other programming languages. R has aspects of both functional and object-oriented (OO) programming languages. Learning how these concepts are expressed in R will help you leverage your existing knowledge of other programming languages, and will help you identify areas where you can improve.

To understand why R's object systems work the way they do, I found [_The Structure and Interpretation of Computer Programs_](https://mitpress.mit.edu/sites/default/files/sicp/full-text/book/book.html) (SICP) by Harold Abelson and Gerald Jay Sussman, particularly helpful. It's a concise but deep book. After reading it, I felt for the first time that I could actually design my own object-oriented system. The book was my first introduction to the generic function style of OO common in R. It helped me understand its strengths and weaknesses. SICP also talks a lot about functional programming, and how to create simple functions which become powerful when combined.

To understand the trade-offs that R has made compared to other programming languages, I found [_Concepts, Techniques and Models of Computer Programming_](http://amzn.com/0262220695?tag=devtools-20) by Peter van Roy and Sef Haridi extremely helpful. It helped me understand that R's copy-on-modify semantics make it substantially easier to reason about code, and that while its current implementation is not particularly efficient, it is a solvable problem.

If you want to learn to be a better programmer, there's no place better to turn than [_The Pragmatic Programmer_](http://amzn.com/020161622X?tag=devtools-20) by Andrew Hunt and David Thomas. This book is language agnostic, and provides great advice for how to be a better programmer.

## Getting help {#getting-help}

Currently, there are three main venues to get help when you're stuck and can't figure out what's causing the problem: [RStudio Community](https://community.rstudio.com/), [stackoverflow](http://stackoverflow.com) and the R-help mailing list. You can get fantastic help in each venue, but they do have their own cultures and expectations. It's usually a good idea to spend a little time lurking, learning about community expectations, before you put up your first post. \index{help}

Some good general advice:

* Make sure you have the latest version of R and of the package (or packages)
  you are having problems with. It may be that your problem is the result of
  a recently fixed bug.

* Spend some time creating a
  [reproducible example](http://stackoverflow.com/questions/5963269). This
  is often a useful process in its own right, because in the course of making
  the problem reproducible you often figure out what's causing the problem. 
  The [`reprex`](https://reprex.tidyverse.org/) package can help you create a 
  **repr**oducible **ex**ample that will can easily be run by people trying to help you. 
  There are [several resources available](https://community.rstudio.com/t/faq-whats-a-reproducible-example-reprex-and-how-do-i-do-one/5219) to help you create a successful `reprex`.

* Look for related problems before posting. If someone has already asked
  your question and it has been answered, it's much faster for everyone if you
  use the existing answer.

## Acknowledgments {#intro-ack}

I would like to thank the tireless contributors to R-help and, more recently, [stackoverflow](http://stackoverflow.com/questions/tagged/r). There are too many to name individually, but I'd particularly like to thank Luke Tierney, John Chambers, Dirk Eddelbuettel, JJ Allaire and Brian Ripley for generously giving their time and correcting my countless misunderstandings.

This book was [written in the open](https://github.com/hadley/adv-r/), and chapters were advertised on [twitter](https://twitter.com/hadleywickham) when complete. It is truly a community effort: many people read drafts, fixed typos, suggested improvements, and contributed content. Without those contributors, the book wouldn't be nearly as good as it is, and I'm deeply grateful for their help. Special thanks go to Peter Li, who read the book from cover-to-cover and provided many fixes. Other outstanding contributors were Aaron Schumacher, \@crtahlin, Lingbing Feng, \@juancentro, and \@johnbaums. \index{contributors}




Thanks go to all contributers in alphabetical order: Aaron Wolen (\@aaronwolen), \@absolutelyNoWarranty, Adam Hunt (\@adamphunt), \@agrabovsky, Alexander Grueneberg (\@agrueneberg), Anthony Damico (\@ajdamico), James Manton (\@ajdm), Aaron Schumacher (\@ajschumacher), Alan Dipert (\@alandipert), Alex Brown (\@alexbbrown), \@alexperrone, Alex Whitworth (\@alexWhitworth), Alexandros Kokkalis (\@alko989), \@amarchin, Amelia McNamara (\@AmeliaMN), Bryce Mecum (\@amoeba), Andrew Laucius (\@andrewla), Andrew Bray (\@andrewpbray), Andrie de Vries (\@andrie), \@aranlunzer, Ari Lamstein (\@arilamstein), \@asnr, Andy Teucher (\@ateucher), Albert Vilella (\@avilella), baptiste (\@baptiste), Brian G. Barkley (\@BarkleyBG), Mara Averick (\@batpigandme), Barbara Borges Ribeiro (\@bborgesr), Brandon Greenwell (\@bgreenwell), Brandon Hurr (\@bhive01), Jason Knight (\@binarybana), Brett Klamer (\@bklamer), Jesse Anderson (\@blindjesse), Brian Mayer (\@blmayer), Benjamin L. Moore (\@blmoore), Brian Diggs (\@BrianDiggs), Brian S. Yandell (\@byandell), \@carey1024, Chip Hogg (\@chiphogg), Chris Muir (\@ChrisMuir), Christopher Gandrud (\@christophergandrud), Clay Ford (\@clayford), Colin Fay (\@ColinFay), \@cortinah, Cameron Plouffe (\@cplouffe), Carson Sievert (\@cpsievert), Craig Citro (\@craigcitro), Craig Grabowski (\@craiggrabowski), Christopher Roach (\@croach), Peter Meilstrup (\@crowding), Crt Ahlin (\@crtahlin), Carlos Scheidegger (\@cscheid), Colin Gillespie (\@csgillespie), Christopher Brown (\@ctbrown), Davor Cubranic (\@cubranic), Darren Cusanovich (\@cusanovich), Christian G. Warden (\@cwarden), Charlotte Wickham (\@cwickham), Dean Attali (\@daattali), Dan Sullivan (\@dan87134), Daniel Barnett (\@daniel-barnett), Kenny Darrell (\@darrkj), Tracy Nance (\@datapixie), Dave Childers (\@davechilders), David Rubinger (\@davidrubinger), David Chudzicki (\@dchudz), Daisuke ICHIKAWA (\@dichika), david kahle (\@dkahle), David LeBauer (\@dlebauer), David Schweizer (\@dlschweizer), David Montaner (\@dmontaner), Zhuoer Dong (\@dongzhuoer), Doug Mitarotonda (\@dougmitarotonda), Jonathan Hill (\@Dripdrop12), Julian During (\@duju211), \@duncanwadsworth, \@eaurele, Dirk Eddelbuettel (\@eddelbuettel), \@EdFineOKL, Edwin Thoen (\@EdwinTh), Ethan Heinzen (\@eheinzen), \@eijoac, Joel Schwartz (\@eipi10), Eric Ronald Legrand (\@elegrand), Ellis Valentiner (\@ellisvalentiner), Emil Hvitfeldt (\@EmilHvitfeldt), Emil Rehnberg (\@EmilRehnberg), Daniel Lee (\@erget), Eric C. Anderson (\@eriqande), Enrico Spinielli (\@espinielli), \@etb, David Hajage (\@eusebe), Fabian Scheipl (\@fabian-s), \@flammy0530, François Michonneau (\@fmichonneau), Francois Pepin (\@fpepin), Frank Farach (\@frankfarach), \@freezby, Frans van Dunné (\@FvD), \@fyears, \@gagnagaman, Garrett Grolemund (\@garrettgman), Gavin Simpson (\@gavinsimpson), \@gezakiss7, \@gggtest, Gökçen Eraslan (\@gokceneraslan), Georg Russ (\@gr650), \@grasshoppermouse, Gregor Thomas (\@gregorp), Garrett See (\@gsee), Ari Friedman (\@gsk3), Gunnlaugur Thor Briem (\@gthb), Hadley Wickham (\@hadley), Hamed (\@hamedbh), Harley Day (\@harleyday), \@hassaad85, \@helmingstay, Henning (\@henningsway), Henrik Bengtsson (\@HenrikBengtsson), Ching Boon (\@hoscb), Iain Dillingham (\@iaindillingham), \@IanKopacka, Ian Lyttle (\@ijlyttle), Ilan Man (\@ilanman), Imanuel Costigan (\@imanuelcostigan), Thomas Bürli (\@initdch), Os Keyes (\@Ironholds), \@irudnyts, i (\@isomorphisms), Irene Steves (\@isteves), Jan Gleixner (\@jan-glx), Jason Asher (\@jasonasher), Jason Davies (\@jasondavies), Chris (\@jastingo), jcborras (\@jcborras), John Blischak (\@jdblischak), \@jeharmse, Lukas Burk (\@jemus42), Jennifer (Jenny) Bryan (\@jennybc), Justin Jent (\@jentjr), Jeston (\@JestonBlu), Jim Hester (\@jimhester), \@JimInNashville, \@jimmyliu2017, Jim Vine (\@jimvine), Jinlong Yang (\@jinlong25), J.J. Allaire (\@jjallaire), \@JMHay, Jochen Van de Velde (\@jochenvdv), Johann Hibschman (\@johannh), John Baumgartner (\@johnbaums), John Horton (\@johnjosephhorton), \@johnthomas12, Jon Calder (\@jonmcalder), Jon Harmon (\@jonthegeek), Julia Gustavsen (\@jooolia), JorneBiccler (\@JorneBiccler), Jeffrey Arnold (\@jrnold), Joyce Robbins (\@jtr13), Juan Manuel Truppia (\@juancentro), Kevin Markham (\@justmarkham), john verzani (\@jverzani), Michael Kane (\@kaneplusplus), Bart Kastermans (\@kasterma), Kevin D'Auria (\@kdauria), Karandeep Singh (\@kdpsingh), Ken Williams (\@kenahoo), Kendon Bell (\@kendonB), Kent Johnson (\@kent37), Kevin Ushey (\@kevinushey), 电线杆 (\@kfeng123), Karl Forner (\@kforner), Kirill Sevastyanenko (\@kirillseva), Brian Knaus (\@knausb), Kirill Müller (\@krlmlr), Kriti Sen Sharma (\@ksens), Kevin Wright (\@kwstat), suo.lawrence.liu@gmail.com (\@Lawrence-Liu), \@ldfmrails, Rachel Severson (\@leighseverson), Laurent Gatto (\@lgatto), C. Jason Liang (\@liangcj), Steve Lianoglou (\@lianos), \@lindbrook, Lingbing Feng (\@Lingbing), Marcel Ramos (\@LiNk-NY), Zhongpeng Lin (\@linzhp), Lionel Henry (\@lionel-), myq (\@lrcg), Luke W Johnston (\@lwjohnst86), Kevin Lynagh (\@lynaghk), Malcolm Barrett (\@malcolmbarrett), \@mannyishere, Matt (\@mattbaggott), Matthew Grogan (\@mattgrogan), \@matthewhillary, Matthieu Gomez (\@matthieugomez), Matt Malin (\@mattmalin), Mauro Lepore (\@maurolepore), Max Ghenis (\@MaxGhenis), Maximilian Held (\@maxheld83), Michal Bojanowski (\@mbojan), Mark Rosenstein (\@mbrmbr), Michael Sumner (\@mdsumner), Jun Mei (\@meijun), merkliopas (\@merkliopas), mfrasco (\@mfrasco), Michael Bach (\@michaelbach), Michael Bishop (\@MichaelMBishop), Michael Buckley (\@michaelmikebuckley), Michael Quinn (\@michaelquinn32), \@miguelmorin, Michael (\@mikekaminsky), Mine Cetinkaya-Rundel (\@mine-cetinkaya-rundel), \@mjsduncan, Mamoun Benghezal (\@MoBeng), Matt Pettis (\@mpettis), Martin Morgan (\@mtmorgan), Guy Dawson (\@Mullefa), Nacho Caballero (\@nachocab), Natalya Rapstine (\@natalya-patrikeeva), Nick Carchedi (\@ncarchedi), Noah Greifer (\@ngreifer), Nicholas Vasile (\@nickv9), Nikos Ignatiadis (\@nignatiadis), Xavier Laviron (\@norival), Nick Pullen (\@nstjhp), Oge Nnadi (\@ogennadi), Oliver Paisley (\@oliverpaisley), Pariksheet Nanda (\@omsai), Øystein Sørensen (\@osorensen), Paul (\@otepoti), Otho Mantegazza (\@othomantegazza), Dewey Dunnington (\@paleolimbot), Parker Abercrombie (\@parkerabercrombie), Patrick Hausmann (\@patperu), Patrick Miller (\@patr1ckm), Patrick Werkmeister (\@Patrick01), \@paulponcet, \@pdb61, Tom Crockett (\@pelotom), \@pengyu, Jeremiah (\@perryjer1), Peter Hickey (\@PeteHaitch), Phil Chalmers (\@philchalmers), Jose Antonio Magaña Mesa (\@picarus), Pierre Casadebaig (\@picasa), Antonio Piccolboni (\@piccolbo), Pierre Roudier (\@pierreroudier), Poor Yorick (\@pooryorick), Marie-Helene Burle (\@prosoitos), Peter Schulam (\@pschulam), John (\@quantbo), Quyu Kong (\@qykong), Ramiro Magno (\@ramiromagno), Ramnath Vaidyanathan (\@ramnathv), Kun Ren (\@renkun-ken), Richard Reeve (\@richardreeve), Richard Cotton (\@richierocks), Robert M Flight (\@rmflight), R. Mark Sharp (\@rmsharp), Robert Krzyzanowski (\@robertzk), \@robiRagan, Romain François (\@romainfrancois), Ross Holmberg (\@rossholmberg), Ricardo Pietrobon (\@rpietro), \@rrunner, Ryan Walker (\@rtwalker), \@rubenfcasal, Rob Weyant (\@rweyant), Rumen Zarev (\@rzarev), Nan Wang (\@sailingwave), \@sbgraves237, Scott Kostyshak (\@scottkosty), Scott Leishman (\@scttl), Sean Hughes (\@seaaan), Sean Anderson (\@seananderson), Sean Carmody (\@seancarmody), Sebastian (\@sebastian-c), Matthew Sedaghatfar (\@sedaghatfar), \@see24, Sven E. Templer (\@setempler), \@sflippl, \@shabbybanks, Steven Pav (\@shabbychef), Shannon Rush (\@shannonrush), S'busiso Mkhondwane (\@sibusiso16), Sigfried Gold (\@Sigfried), Simon O'Hanlon (\@simonohanlon101), Simon Potter (\@sjp), Steve (\@SplashDance), Scott Ritchie (\@sritchie73), Tim Cole (\@statist7), \@ste-fan, \@stephens999, Steve Walker (\@stevencarlislewalker), Stefan Widgren (\@stewid), Homer Strong (\@strongh), Dirk (\@surmann), Sebastien Vigneau (\@svigneau), Scott Warchal (\@Swarchal), Steven Nydick (\@swnydick), Taekyun Kim (\@taekyunk), Tal Galili (\@talgalili), \@Tazinho, Tom B (\@tbuckl), \@tdenes, \@thomasherbig, Thomas (\@thomaskern), Thomas Lin Pedersen (\@thomasp85), Thomas Zumbrunn (\@thomaszumbrunn), Tim Waterhouse (\@timwaterhouse), TJ Mahr (\@tjmahr), Anton Antonov (\@tonytonov), Ben Torvaney (\@Torvaney), Jeff Allen (\@trestletech), Terence Teo (\@tteo), Tim Triche, Jr. (\@ttriche), \@tyhenkaline, Tyler Ritchie (\@tylerritchie), Varun Agrawal (\@varun729), Vijay Barve (\@vijaybarve), Victor (\@vkryukov), Vaidotas Zemlys-Balevičius (\@vzemlys), Winston Chang (\@wch), Linda Chin (\@wchi144), Welliton Souza (\@Welliton309), Gregg Whitworth (\@whitwort), Will Beasley (\@wibeasley), William R Bauer (\@WilCrofter), William Doane (\@WilDoane), Sean Wilkinson (\@wilkinson), Christof Winter (\@winterschlaefer), Bill Carver (\@wmc3), Wolfgang Huber (\@wolfganghuber), Krishna Sankar (\@xsankar), Yihui Xie (\@yihui), yang (\@yiluheihei), Yoni Ben-Meshulam (\@yoni), \@yuchouchen, \@zachcp, \@zackham, Edward Cho (\@zerokarmaleft), Albert Zhao (\@zxzb).

## Conventions {#conventions}
\index{website}

Throughout this book I use `f()` to refer to functions, `g` to refer to variables and function parameters, and `h/` to paths. 

Larger code blocks intermingle input and output. Output is commented so that if you have an electronic version of the book, e.g., <https://adv-r.hadley.nz/>, you can easily copy and paste examples into R. Output comments look like `#>` to distinguish them from regular comments. 

Many examples use random numbers. These are made reproducible by `set.seed(1014)` which is run at the start of each chapter.

## Colophon {#colophon}

This book was written in [bookdown](http://bookdown.org/) inside [RStudio](http://www.rstudio.com/ide/). The [website](https://adv-r.hadley.nz/) is hosted with [netlify](http://netlify.com/), and automatically updated after every commit by [travis-ci](https://travis-ci.org/). The complete source is available from [github](https://github.com/hadley/adv-r).

Code in the printed book is set in [inconsolata](http://levien.com/type/myfonts/inconsolata.html).


setting    value                        
---------  -----------------------------
version    R version 3.5.1 (2018-07-02) 
os         OS X El Capitan 10.11.6      
system     x86_64, darwin15.6.0         
ui         X11                          
language   (EN)                         
collate    en_US.UTF-8                  
tz         America/New_York             
date       2018-09-03                   


package          version      source                            
---------------  -----------  ----------------------------------
assertthat       0.2.0        CRAN (R 3.5.0)                    
backports        1.1.2        CRAN (R 3.5.0)                    
base64enc        0.1-3        CRAN (R 3.5.0)                    
BH               1.66.0-1     CRAN (R 3.5.0)                    
bindr            0.1.1        CRAN (R 3.5.0)                    
bindrcpp         0.2.2        CRAN (R 3.5.0)                    
bit              1.1-14       CRAN (R 3.5.0)                    
bit64            0.9-7        CRAN (R 3.5.0)                    
bitops           1.0-6        CRAN (R 3.5.0)                    
blob             1.1.1        CRAN (R 3.5.0)                    
bookdown         0.7          CRAN (R 3.5.0)                    
boot             1.3-20       CRAN (R 3.5.1)                    
caTools          1.17.1.1     CRAN (R 3.5.0)                    
class            7.3-14       CRAN (R 3.5.1)                    
cli              1.0.0        CRAN (R 3.5.0)                    
clisymbols       1.2.0        CRAN (R 3.5.0)                    
cluster          2.0.7-1      CRAN (R 3.5.1)                    
codetools        0.2-15       CRAN (R 3.5.1)                    
colorspace       1.3-2        CRAN (R 3.5.0)                    
crayon           1.3.4        CRAN (R 3.5.0)                    
curl             3.2          CRAN (R 3.5.0)                    
DBI              1.0.0        CRAN (R 3.5.0)                    
dbplyr           1.2.2        CRAN (R 3.5.0)                    
devtools         1.13.6       CRAN (R 3.5.0)                    
digest           0.6.16       CRAN (R 3.5.0)                    
dplyr            0.7.6        CRAN (R 3.5.1)                    
emo              0.0.0.9000   Github (hadley/emo\@02a5206)      
evaluate         0.11         CRAN (R 3.5.0)                    
fansi            0.3.0        CRAN (R 3.5.0)                    
foreign          0.8-70       CRAN (R 3.5.1)                    
ggplot2          3.0.0        CRAN (R 3.5.0)                    
git2r            0.23.0       CRAN (R 3.5.0)                    
glue             1.3.0        Github (tidyverse/glue\@4e74901)  
gtable           0.2.0        CRAN (R 3.5.0)                    
highr            0.7          CRAN (R 3.5.0)                    
hms              0.4.2        CRAN (R 3.5.0)                    
htmltools        0.3.6        CRAN (R 3.5.0)                    
httpuv           1.4.5        cran (\@1.4.5)                    
httr             1.3.1        CRAN (R 3.5.0)                    
jsonlite         1.5          CRAN (R 3.5.0)                    
KernSmooth       2.23-15      CRAN (R 3.5.1)                    
knitr            1.20         CRAN (R 3.5.0)                    
labeling         0.3          CRAN (R 3.5.0)                    
later            0.7.4        cran (\@0.7.4)                    
lattice          0.20-35      CRAN (R 3.5.1)                    
lazyeval         0.2.1        CRAN (R 3.5.0)                    
lineprof         0.1.9001     Github (hadley/lineprof\@972e71d) 
lobstr           0.0.0.9000   Github (hadley/lobstr\@530db70)   
lubridate        1.7.4        cran (\@1.7.4)                    
magrittr         1.5          CRAN (R 3.5.0)                    
markdown         0.8          CRAN (R 3.5.0)                    
MASS             7.3-50       CRAN (R 3.5.1)                    
Matrix           1.2-14       CRAN (R 3.5.1)                    
memoise          1.1.0        CRAN (R 3.5.0)                    
mgcv             1.8-24       CRAN (R 3.5.1)                    
microbenchmark   1.4-4        CRAN (R 3.5.0)                    
mime             0.5          CRAN (R 3.5.0)                    
munsell          0.5.0        CRAN (R 3.5.0)                    
nlme             3.1-137      CRAN (R 3.5.1)                    
nnet             7.3-12       CRAN (R 3.5.1)                    
openssl          1.0.2        CRAN (R 3.5.0)                    
pillar           1.3.0        CRAN (R 3.5.0)                    
pkgconfig        2.0.2        CRAN (R 3.5.0)                    
plogr            0.2.0        CRAN (R 3.5.0)                    
plyr             1.8.4        CRAN (R 3.5.0)                    
praise           1.0.0        CRAN (R 3.5.0)                    
prettyunits      1.0.2        CRAN (R 3.5.0)                    
promises         1.0.1        cran (\@1.0.1)                    
pryr             0.1.4        CRAN (R 3.5.0)                    
purrr            0.2.5        CRAN (R 3.5.0)                    
R6               2.2.2        CRAN (R 3.5.0)                    
RColorBrewer     1.1-2        CRAN (R 3.5.0)                    
Rcpp             0.12.18      CRAN (R 3.5.0)                    
readr            1.1.1        CRAN (R 3.5.0)                    
reshape2         1.4.3        CRAN (R 3.5.0)                    
rlang            0.2.2.9000   Github (r-lib/rlang\@9a1fc75)     
rmarkdown        1.10         CRAN (R 3.5.0)                    
rpart            4.1-13       CRAN (R 3.5.1)                    
rprojroot        1.3-2        CRAN (R 3.5.0)                    
RSQLite          2.1.1        CRAN (R 3.5.0)                    
rstudioapi       0.7          CRAN (R 3.5.0)                    
scales           1.0.0        CRAN (R 3.5.0)                    
sessioninfo      1.0.0        CRAN (R 3.5.0)                    
shiny            1.1.0        cran (\@1.1.0)                    
sloop            0.0.0.9000   Github (hadley/sloop\@ece21d8)    
sourcetools      0.1.7        cran (\@0.1.7)                    
spatial          7.3-11       CRAN (R 3.5.1)                    
stringi          1.2.4        CRAN (R 3.5.0)                    
stringr          1.3.1        CRAN (R 3.5.0)                    
survival         2.42-3       CRAN (R 3.5.1)                    
testthat         2.0.0        CRAN (R 3.5.0)                    
tibble           1.4.2        CRAN (R 3.5.0)                    
tidyselect       0.2.4        CRAN (R 3.5.0)                    
tinytex          0.8          CRAN (R 3.5.0)                    
utf8             1.1.4        CRAN (R 3.5.0)                    
viridisLite      0.3.0        CRAN (R 3.5.0)                    
whisker          0.3-2        CRAN (R 3.5.0)                    
withr            2.1.2        CRAN (R 3.5.0)                    
xfun             0.3          CRAN (R 3.5.0)                    
xtable           1.8-3        cran (\@1.8-3)                    
yaml             2.2.0        CRAN (R 3.5.0)                    
zeallot          0.1.0        CRAN (R 3.5.0)                    




```r
ruler()
#> ----+----1----+----2----+----3----+----4----+----5----+----6----+----7----+
#> 123456789012345678901234567890123456789012345678901234567890123456789012345
```

*[I originally wrote this for [Perlmonks](https://www.perlmonks.org/?node_id=376075), and then included it as an appendix in [Mastering Perl](http://www.masteringperl.org).*

![](https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png) This work is licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-nc-sa/4.0/).

## My Philosophy of Debugging

I believe in three things:

1. It is not personal

Forget about code ownership. You may think yourself an artist, but
even the old Masters produced a lot of crap. Everybody's code is
crap, which means my code is crap and your code is crap. Learn to
love that. When you have a problem, your first thought should be
"Something is wrong with my crappy code". That means you do not get
to blame perl. It is not personal.

Forget about how *you* do things. If the way you did things worked,
you would not be reading this. That is not a bad thing. It is just
time to evolve. We have all been there.

2. Personal responsibility

If you have a problem with your script it is just that—your problem.
You should do as much to solve it by yourself as you can. Remember,
everyone else has their own scripts, which means they have their own
problems. Do your homework and give it your best shot before you
bother someone else with your problems. If you honestly try
everything in this guide and still cannot solve the problem, you have
given it your best shot and it is time to bother someone else.

3. Change how you do things

Fix things so you do not have the same problem again. The problem is
probably *how* you code, not *what* you code. Change the way you do
things to make your life easier. Do not make Perl adapt to you
because it will not. Adapt to Perl. It is just a language, not a way
of life.

## My Method

### Does your script compile with strictures?

If you are not already using strictures, turn it on. Perl gurus are gurus because they use strict which leaves them more time to solve other problems, learn new things, and upload working modules to CPAN.

You can turn on strictures within the code with the strict pragma.

    use strict;

You can turn on strictures from the command line with perl's -M switch.

    % perl -Mstrict script.pl

With versions of Perl since v5.12, specifying the version with `use` automatically turns on `strict`:

	use v5.30;

You'll likely have to fix a bunch of stuff at first, but if there's a part of your code you can't get past `strict`, you can turn it off within a block of code so you can deal with it later:

	{
	no strict;

	...
	}

You may be annoyed at strictures, but after a couple of weeks of programming with them turned on, you will write better code, spend less time chasing simple errors, and probably will not need this guide.

### What is the warning?

Perl will warn you about a lot of questionable constructs. Turn on warnings and help Perl help you. Do this before you present code to other people so save yourself some embarrassment and to respect other people's time and generosity.

You can use perl's `-w` switch in the shebang line to turn on warnings everywhere in your program:

    #!/usr/bin/perl -w

You can turn on `warnings` from the command line:

    % perl -w script.pl

You can use lexical warnings with all sorts of interesting features. See [warnings](https://perldoc.perl.org/warnings.html) for the details.

    use warnings;

If you do not understand a warning, you can look up a verbose version of the warning in [perldiag](https://perldoc.perl.org/perldiag.html) or you can use the diagnostics pragma in your code.

    use diagnostics;

### Solve the first problem first!

After you get error or warning messages from `perl`, fix the first message then see if the `perl` still issues the other messages. Those extra messages may be artifacts of the first problem.

### Look at the code before the line number in the error message!

Perl gives you warning messages when it gets worried and not before. By the time perl gets worried the problem has already occurred and the line number perl is on is actually *after* the problem. Look at the couple of expressions before the line number in the warning.

### Is the value what you think it is?

Do not guess! Actually examine the value right before you want to use it in an expression. The best debugger in the universe is [print](https://perldoc.perl.org/functions/print.html).

    print STDERR "The value is [$value]";

I enclose `$value` in braces so I can see any leading or trailing whitespace or newlines.

If I have anything other than a scalar, I use [Data::Dumper](https://metacpan.org/pod/Data::Dumper) to print the data structures.

    require Data::Dumper;

    print STDERR "The hash is ",
    	Data::Dumper::Dumper( \%hash ),
    	"\n";

If the value is not what you think it is, back up a few steps and try again! Do this until you find the point at which the value stops being what you think it should be!

You can also use the built-in perl debugger with perl's `-d` switch. See [perldebug](https://perldoc.perl.org/perldebug.html) for details.

    % perl -d script.pl

You can also use other debuggers or development environments, like a [ptkdb](https://metacpan.org/pod/Devel::ptkdb) (a graphical debugger based on Tk) or [Komodo](https://www.activestate.com/products/komodo-ide/perl-editor/) (ActiveState's Perl IDE based on Mozilla).

### Are you using the function correctly?

I have been programming Perl for quite a long time and I still look at [perlfunc](https://perldoc.perl.org/perlfunc.html) almost every day. Some things I just cannot keep straight, and sometimes I am so sleep-deprived that I take leave of all of my senses and wonder why [sprintf](https://perldoc.perl.org/functions/sprintf.html) does not print to the screen.

You can look up a particular function with the perldoc command and its
`-f` switch.

    % perldoc -f function_name

If you are using a module, check the documentation to make sure you
are using it in the right way. You can check the documentation for
the module using `perldoc`.

    % perldoc Module::Name

### Are you using the right special variable?

Again, I constantly refer to [perlvar](https://perldoc.perl.org/perlvar.html). Well, not really since I find [The Perl Pocket Reference](https://amzn.to/2VBj4sY) much easier to use. There's also the [Perl Quick Reference Card](http://johnbokma.com/perl/perl-quick-reference-card.pdf), although it doesn't list the special variables.

### Do you have the right version of the module?

Some modules change behavior between versions. Do you have the version of the module that you think you have? You can check the module version with a simple perl one-liner.

    % perl -MModule::Name -le 'print Module::Name->VERSION';

If you read most of your documentation off of the local machine, like at [https://perldoc.perl.org](https://perldoc.perl.org) or [MetaCPAN](https://metacpan.org/) then you are more likely to encounter version differences in documentation.

### Have you made a small test case?

If you are trying something new, or think a particular piece of code is acting funny, write the shortest possible program to do just that piece. This removes most of the other factors from consideration. If the small test program does what it thinks it does, the problem probably is not in that code. If the program does not do what you think it does, then perhaps you have found your problem.

### Did you check the environment?

Some things depend on environment variables. Are you sure that they are set to the right thing? Is your environment the same that the program will see when it runs? Remember that programs intended for CGI programs or cron jobs may see different environments than those in your interactive shell, especially on different machines.

Perl stores the environment in `%ENV`. If you need one of those variables, be ready to supply a default value if it does not exist, even if only for testing.

If you still have trouble, inspect the environment.

    require Data::Dumper;
    print STDERR Data::Dumper::Dumper( \%ENV );

### Have you checked Google?

If you have a problem, somebody else has probably had that problem. See if one of those other people posted something about it. When I originally wrote this, Usenet and Google Groups were the hot things. Now it's Google leading to [StackOverflow](https://www.stackoverflow.com) answers.

Often I'll paste the literal error message into Google and find that someone else has posted that same error message. Many times, I don't find something useful right away so I have to dig a bit.

### Have you profiled the application?

If you want to track down the slow parts of the program, have you profiled it? Let [Devel::SmallProf](https://metacpan.org/pod/Devel::SmallProf) do the heavy lifting for you. It counts the times perl executes a line of code as well as how long it takes and prints a nice report.

[NYTProf](https://metacpan.org/pod/Devel::NYTProf) is a heavyweight solution that collects lots of information and makes nice plots. It's something you'll eventually should use for big applications, but if you haven't started using it you might not want to slow down to learn it immediately.

### Which test fails?

If you have a test suite, which test fails? You should be able to track down the error very quickly since each test will only exercise a little bit of code.

If you don't have a test suite, why not make one? If you have a really small script, or this is a one-off script, then I will not make you write a couple of tests. Anything other than that could really benefit from some test scripts. [Test::Harness](https://metacpan.org/pod/Test::Harness) makes this so simple that you really have no excuse not to do it. If you do not have the time, perhaps you are wasting too much time debugging scripts without tests. [MakeMaker](https://metacpan.org/pod/ExtUtils::MakeMaker) is just not for modules after all.

### Did you talk to the bear?

Explain your problem aloud. Actually say the words aloud. At first you may think this is stupid, but the idea in your head only seems fully formed until you try to verbalize. It's then that you have to fill in the detail that you only imagined were there.

For a couple of years I had the pleasure of working with a really good programmer who could solve almost anything. When I got really stuck I would walk over to his desk and start to explain my problem. Usually I didn't make it past the third sentence without saying "Never mind—I got it". He almost never missed either.

Since you will probably need to do this so much, I recommend some sort of plush toy to act as your Perl therapist so you do not annoy your colleagues. I have a small bear that sits on my desk and I explain problems to him. My wife does not even pay attention when I talk to myself anymore.

### Does the problem look different on paper?

You have been staring at the computer screen, so maybe a different medium will let you look at things in a new way. Try looking at a print-out of your program.

If you don't want to use dead trees, try looking at the source without syntax highlight, or even in a different editor. Change up the way it looks and the fonts you use so you see different patterns.

### Try working on something else

There are probably other things you need to work on within the same  project. Perhaps paying attention to other items in your to-do list gives you some ideas.

### Have you watched *Last Week Tonight*?

Seriously. Perhaps you do not like John Oliver, so choose something else. Take a break. Stop thinking about the problem for a bit and let your mind relax. Come back to the problem later and the fix may become immediately apparent.

You don't even to do something active. Many people find that taking a shower or sleeping on it magically gives them the answer.

### Have you packed your ego?

If you still have not made it this far, the problem may be psychological. You might be emotionally attached to a certain part of the code, so you do not change it. You might also think that everyone else is wrong but you. When you do that, you do not seriously consider the most likely source of bugs—yourself. Do not ignore anything. Verify everything.

---
tags: programming
---

# Comparison of C++ Command Line Interface (CLI) parsers

So given how many years C++ has on its back, you would say it should have well-thought libraries,
built with the pain-hardened experience of failed attemps, right? As much as a C++ fanboy I am, I'm
afraid that's not always (or not often?) the case.

Here I'm going to summarize the good and bad design ideas of some CLI parsers I got my hands on.

# Getopt

I guess this is the good old cli parser that came from C era.

Complete example taken from [official
docs](https://www.gnu.org/software/libc/manual/html_node/Getopt-Long-Option-Example.html#Getopt-Long-Option-Example):

```c
#include <stdio.h>
#include <stdlib.h>
#include <getopt.h>

/* Flag set by ‘--verbose’. */
static int verbose_flag;

int
main (int argc, char **argv)
{
  int c;

  while (1)
    {
      static struct option long_options[] =
        {
          /* These options set a flag. */
          {"verbose", no_argument,       &verbose_flag, 1},
          {"brief",   no_argument,       &verbose_flag, 0},
          /* These options don’t set a flag.
             We distinguish them by their indices. */
          {"add",     no_argument,       0, 'a'},
          {"append",  no_argument,       0, 'b'},
          {"delete",  required_argument, 0, 'd'},
          {"create",  required_argument, 0, 'c'},
          {"file",    required_argument, 0, 'f'},
          {0, 0, 0, 0}
        };
      /* getopt_long stores the option index here. */
      int option_index = 0;

      c = getopt_long (argc, argv, "abc:d:f:",
                       long_options, &option_index);

      /* Detect the end of the options. */
      if (c == -1)
        break;

      switch (c)
        {
        case 0:
          /* If this option set a flag, do nothing else now. */
          if (long_options[option_index].flag != 0)
            break;
          printf ("option %s", long_options[option_index].name);
          if (optarg)
            printf (" with arg %s", optarg);
          printf ("\n");
          break;

        case 'a':
          puts ("option -a\n");
          break;

        case 'b':
          puts ("option -b\n");
          break;

        case 'c':
          printf ("option -c with value `%s'\n", optarg);
          break;

        case 'd':
          printf ("option -d with value `%s'\n", optarg);
          break;

        case 'f':
          printf ("option -f with value `%s'\n", optarg);
          break;

        case '?':
          /* getopt_long already printed an error message. */
          break;

        default:
          abort ();
        }
    }

  /* Instead of reporting ‘--verbose’
     and ‘--brief’ as they are encountered,
     we report the final status resulting from them. */
  if (verbose_flag)
    puts ("verbose flag is set");

  /* Print any remaining command line arguments (not options). */
  if (optind < argc)
    {
      printf ("non-option ARGV-elements: ");
      while (optind < argc)
        printf ("%s ", argv[optind++]);
      putchar ('\n');
    }

  exit (0);
}
```

Well, that was not beautiful...

## Good ideas

### Interesting flag handling 

`{"verbose", no_argument,       &verbose_flag, 1},` means that if
`--verbose` is provided, it will "automatically" fill the variable `verbose_flag` with that last
parameter `1`.

### Powerful (?)

When you are the one on charge of doing things, only your laziness is the limit! It 
allows flags, parameters with arguments (required or not) and standalone arguments; you can do
whatever you want with them.

### Teaching good practices (?)

> Some hours of trial and error can save you some minutes of reading documentation.

This is also a downside, but this library teaches you to look for the documentation as if it were
air to breathe. When you first encounter an example of usage of this library without reading the
documentation, there are so many questions, like:

- What do those 0s mean in the `struct options long_options`? Do the 0s in the 4th column, the 0s in
  the 3rd column, and the 0s in the last row mean the same?

- What the heck is this string "abc:d:f:"?

- Why `c` is used to stop the loop, as char in some `switch` cases, and as 0 in other cases? And why
  there's a `case '?':` if it's not an option in `long_options`?

Those are not complex concepts once you read the docs, and noob programmers should learn soon that
docs are essential to programming, but honestly, good code should be low on WTFs per minute, even
when you didn't start reading the docs yet.


## Bad ideas

### That fat-ass `switch`.

You know, `switch` is not the best feature of programming languages, especially old ones.

- Will the compiler tell you that you forgot to handle some parameter? No.

- Will it allow you to use parameters with long options only (without the one letter parameter name)? No,
because `switch`es in C/C++ don't work with strings. `char`s seem even a hack. You have to add a `case
0:` and check `long_options[option_index].name` to know which parameter are you handling.

### Global hidden state.

It's a `switch` within a `while(1)`, so each iteration will go through one of the cases of the
`switch`. The parameter you can handle in each iteration is provided by `c = getopt_long (argc,
argv, "abc:d:f:", long_options, &option_index);`. `c` is the one-char parameter name, and
`option_index` is modified to point to a row in `long_options`. So each time you call that function
(with the same parameters) you will get a different result.

Moreover, this is not all the information needed. The library provides some **global variables** to pass
parameter values, unexpected input, further details and error codes.

## Getopt Summary

To be fair, with plain C these issues are quite unavoidable, it's an OK solution for those
early days, but nowadays we should know better.


# TCLAP

Again, [official example](http://tclap.sourceforge.net/manual.html#BASIC_USAGE):

```c++
#include <string>
#include <iostream>
#include <algorithm>
#include <tclap/CmdLine.h>

int main(int argc, char** argv)
{

	// Wrap everything in a try block.  Do this every time, 
	// because exceptions will be thrown for problems.
	try {  

	// Define the command line object, and insert a message
	// that describes the program. The "Command description message" 
	// is printed last in the help text. The second argument is the 
	// delimiter (usually space) and the last one is the version number. 
	// The CmdLine object parses the argv array based on the Arg objects
	// that it contains. 
	TCLAP::CmdLine cmd("Command description message", ' ', "0.9");

	// Define a value argument and add it to the command line.
	// A value arg defines a flag and a type of value that it expects,
	// such as "-n Bishop".
	TCLAP::ValueArg<std::string> nameArg("n","name","Name to print",true,"homer","string");

	// Add the argument nameArg to the CmdLine object. The CmdLine object
	// uses this Arg to parse the command line.
	cmd.add( nameArg );

	// Define a switch and add it to the command line.
	// A switch arg is a boolean argument and only defines a flag that
	// indicates true or false.  In this example the SwitchArg adds itself
	// to the CmdLine object as part of the constructor.  This eliminates
	// the need to call the cmd.add() method.  All args have support in
	// their constructors to add themselves directly to the CmdLine object.
	// It doesn't matter which idiom you choose, they accomplish the same thing.
	TCLAP::SwitchArg reverseSwitch("r","reverse","Print name backwards", cmd, false);

	// Parse the argv array.
	cmd.parse( argc, argv );

	// Get the value parsed by each arg. 
	std::string name = nameArg.getValue();
	bool reverseName = reverseSwitch.getValue();

	// Do what you intend. 
	if ( reverseName )
	{
		std::reverse(name.begin(),name.end());
		std::cout << "My name (spelled backwards) is: " << name << std::endl;
	}
	else
		std::cout << "My name is: " << name << std::endl;


	} catch (TCLAP::ArgException &e)  // catch any exceptions
	{ std::cerr << "error: " << e.error() << " for arg " << e.argId() << std::endl; }
}
```

## Good ideas

### Easier handling

The API seems easier to understand. We declare a `cmd` that will hold the parameters, we declare a
`<variable>Arg` with the configuration of each parameter, we parse the `cmd`, and then we get the
results into the corresponding variables. Also catching any error with exceptions.

In fact, the next lines are doing the same as the previous `switch` in getopt:
```c++
	std::string name = nameArg.getValue();
	bool reverseName = reverseSwitch.getValue();
```

### Type safety

With getopt all we had were strings, but now we have templates and for basic types the parsing is
done automatically. 

### Good automatic help

The example doesn't provide the `--help` parameter, but it will work as expected if it's used. This
is a good design in my opinion. Providing sensible defaults so that you only need to add code when
you want to do uncommon things. I would say python and the spring framework follow this idea.

It even asks for a human readable string to explain what type is expected, which I have used
sometimes to put the units and/or range of the parameter, e.g. "seconds (up to 3600)" in parameter:

```c++
    TCLAP::ValueArg<double> durationArg("d", names[DURATION], "duration of the song",
            false, defaultDuration, "seconds (up to 3600)", cmd);
```
which is shown in the help as:

```
-d <seconds (up to 3600)>, --duration <seconds (up to 3600)>
    duration of the song
```

Not bad as default help message.

## Bad ideas

### Not showing default values

The reason why I stopped using TCLAP was not being able to show the default value I provided. In the
previous `durationArg` parameter example, I used `double defaultDuration = 20;`, but I found no easy
way to show in the help message this "20".

I could build the description string with the variable like `"duration of the song (default " +
std::to_string(defaultDuration) + ")"`, but it seems unreasonably complex for such a relevant
feature in a CLI parser. 

I also could have hardcoded the value in the description string, but I have a condition that
prevents me from that: my stomach hurts when one conceptual change involves changing several parts
of the code.

### Too many parameters

For most parameter definitions, it feels that some variables are just boilerplate. Have you ever
used a command where all the parameters were required? Then, why not making them optional by
default? Instead, TCLAP requires that `false` (or `true`) in all parameters.

Also, adding the `cmd` parameter seems like could be avoided with other design, although this might
be more difficult, as it was clearly in favour of direct access to the Arg variables (such as
`durationArg` in my example).

## TCLAP summary

Not bad, not perfect, easy to integrate (only one header) and fast to learn and use. You can tweak
some things, like how the help message is written, but it's not so easy and non-trivial work is
needed.

# Boost.Program_options

[Official example](http://www.boost.org/doc/libs/1_66_0/doc/html/program_options.html), complete
code in
[github](https://github.com/boostorg/program_options/blob/5a85b81fcfbd2e3b6ea0522885128a1ccf1d354b/example/first.cpp):

```c++
// Copyright Vladimir Prus 2002-2004.
// Distributed under the Boost Software License, Version 1.0.
// (See accompanying file LICENSE_1_0.txt
// or copy at http://www.boost.org/LICENSE_1_0.txt)

/* The simplest usage of the library.
 */

#include <boost/program_options.hpp>
namespace po = boost::program_options;

#include <iostream>
#include <iterator>
using namespace std;

int main(int ac, char* av[])
{
    try {

        po::options_description desc("Allowed options");
        desc.add_options()
            ("help", "produce help message")
            ("compression", po::value<double>(), "set compression level")
        ;

        po::variables_map vm;        
        po::store(po::parse_command_line(ac, av, desc), vm);
        po::notify(vm);    

        if (vm.count("help")) {
            cout << desc << "\n";
            return 0;
        }

        if (vm.count("compression")) {
            cout << "Compression level was set to " 
                 << vm["compression"].as<double>() << ".\n";
        } else {
            cout << "Compression level was not set.\n";
        }
    }
    catch(exception& e) {
        cerr << "error: " << e.what() << "\n";
        return 1;
    }
    catch(...) {
        cerr << "Exception of unknown type!\n";
    }

    return 0;
}
```


## Good ideas

### Terse parameter configuration

Here we can see basic configuration in action, working with sensible defaults.

If we want to make a parameter required, we can add it `po::value<double>()->required()`.

If we want to add default values, we can add it:
`po::value<double>()->default_value(defaultCompression)`.

If we want to fill our variable directly, we can add it: `po::value<double>(&compression)`.

### Flexibility

With Program Options we don't need a dedicated variable, we can use `vm[].count()` and/or
`vm[].as<>()`, but we are also able to use a variable directly as `po::value<double>(&compression)`
and not use the `count` nor `as`. I saw some other minor flexibility features.

### Showing default values (yes!)

The next parameter:
```c++
desc.add_options()
        ("duration,d",
                po::value<double>(&duration)->default_value(defaultDuration),
                "duration in seconds (up to 3600)")
```

is shown as:
```
  -d [ --duration ] arg (=20)           duration in seconds (up to 3600)
```

I didn't see an easy way to put the value description (such as units) instead of that
not-so-clear "arg", so I had to put that in the description, but it did show that "20" was the
default value, without configuring anything.

### More features

Although I haven't dug in the details, Program Options seems more complete than TCLAP. For instance,
PO allows taking parameters from config files and merge the parameters with the CLI ones.

## Bad ideas

### Expert friendly

Most Boost libraries err a bit on the side of making the library too difficult to use even in the
basic use case. Compare the 4 straightforward steps of TCLAP:

```
    TCLAP::CmdLine cmd(...);
    TCLAP::SwitchArg reverseSwitch(..., cmd);
    cmd.parse( argc, argv );
    bool reverseName = reverseSwitch.getValue();
```
with the PO workflow:
```
    po::options_description desc(...);
    desc.add_options()
        ("compression", po::value...)
    ;

    po::variables_map vm;
    po::store(po::parse_command_line(ac, av, desc), vm);
    po::notify(vm);

    // use vm["compression"], or the variable compression if set with po::value<double>(&compression)
```

### On the edge of abusing operator overload

Operator overloading is like a "pay what you can" store. It's cool and less cumbersome, but it's too
easy to abuse the system.

Once you understand that the `operator()` in `desc.add_options()(...)` is only constructing an
option and adding it to the `options_description`, it's not a big deal, but wouldn't it be more
intuitive a `desc.add_options().add(...)`? it's 4 characters, and it would fit common ideas like the
design pattern [builder](https://en.wikipedia.org/wiki/Builder_pattern).

### Encourages char arrays instead of std::string

I agree this is not a big issue, but it's been 2 projects already when I found this a bit annoying.
The `operator()` asks only for `char*` and not `std::string`, so if you need to compose the strings
for the parameter names or the description, you will have to use either char arrays or lots of
`.c_str()`. This is how it might look.

```c++
desc.add_options()
        ((names[DURATION] + ",d").c_str(),
                po::value<double>(&duration)->default_value(defaultDuration),
                ("duration in seconds (up to " + std::to_string(MAX_DURATION) + ")").c_str())
```

## Program Options summary

The bazooka solution. It will require other parts of Boost. Read the documentation with no hurries
and hope you don't need to tweak it.

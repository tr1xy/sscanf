
# sscanf 2.9.0

## NPC modes

To use sscanf in an NPC mode, download this file:

http://dl.dropbox.com/u/21683085/npcdll.rar

And extract it to your root server directory (so `amxsscanf.dll` is in the same directory as `samp-npc.exe`). Then use as normal. The only tiny difference is that `u`, `r`, and `q` don't know if a user is a bot or not, so just assume they are all players - use accordingly.

## Downloads

GitHub repo:

https://github.com/Y-Less/sscanf/

## Use

This behaves exactly as the old sscanf did, just MUCH faster and much more flexibly. To use it add:

[pawn]
#include <sscanf2>
[/pawn]

To your modes and remove the old sscanf (the new include will detect the old version and throw an error if it is detected). On windows add:

[pawn]
plugins sscanf
[/pawn]

To server.cfg. On Linux add:

[pawn]
plugins sscanf.so
[/pawn]

The basic code looks like:

[pawn]
if (sscanf(params, "ui", giveplayerid, amount))
{
    return SendClientMessage(playerid, 0xFF0000AA, "Usage: /givecash <playerid/name> <amount>");
}
[/pawn]

However it should be noted that sscanf can be used for any text processing you like. For example an ini processor could look like (don't worry about what the bits mean at this stage):

[pawn]
if (sscanf(szFileLine, "p<=>s[8]s[32]", szIniName, szIniValue))
{
    printf("Invalid INI format line");
}
[/pawn]

There is also an alternate function name to avoid confusion with the C standard sscanf:

[pawn]
if (unformat(params, "ui", giveplayerid, amount))
{
    return SendClientMessage(playerid, 0xFF0000AA, "Usage: /givecash <playerid/name> <amount>");
}
[/pawn]

## Specifiers

The available specifiers (the letters `u`, `i` and `s` in the codes above) are below.

    [LIST]
[*][b][color=red][u][color=green]Basic specifiers[/color][/u][/color][/b][/LIST]

[code]
[b][u]Specifier(s)[/u][/b]			[b][u]Name[/u][/b]				[b][u]Example values[/u][/b]
	i, d			Integer				1, 42, -10
	c			Character			a, o, *
	l			Logical				true, false
	b			Binary				01001, 0b1100
	h, x			Hex				1A, 0x23
	o			Octal				045 12
	n			Number				42, 0b010, 0xAC, 045
	f			Float				0.7, -99.5
	g			IEEE Float			0.7, -99.5, INFINITY, -INFINITY, NAN, NAN_E
	u			User name/id (bots and players)	Y_Less, 0
	q			Bot name/id			ShopBot, 27
	r			Player name/id			Y_Less, 42
[/code]

    [LIST][*][b][color=red][u][color=green]Strings[/color][/u][/color][/b][/LIST]


The specifier `s` is used, as before, for strings - but they are now more advanced. As before they support collection, so doing:

[pawn]
sscanf("hello 27", "si", str, val);
[/pawn]

Will give:

[code]
hello
27
[/code]

Doing:

[pawn]
sscanf("hello there 27", "si", str, val);
[/pawn]

Will fail as `there` is not a number. However doing:

[pawn]
sscanf("hello there", "s", str);
[/pawn]

Will give:

[code]
hello there
[/code]

Because there is nothing after `s` in the specifier, the string gets everything. To stop this simply add a space:

[pawn]
sscanf("hello there", "s ", str);
[/pawn]

Will give:

[code]
hello
[/code]

You can also escape parts of strings with `\\` - note that this is two backslashes as 1 is used by the compiler:

[pawn]
sscanf("hello\\ there 27", "si", str, val);
[/pawn]

Will give:

[code]
hello there
27
[/code]

All these examples however will give warnings in the server as the new version has array sizes. The above code should be:

[pawn]
new
    str[32],
    val;
sscanf("hello\\ there 27", "s[32]i", str, val);
[/pawn]

As you can see - the format specifier now contains the length of the target string, ensuring that you can never have your strings overflow and cause problems. This can be combined with the SA:MP compiler's stringizing:

[pawn]
#define STR_SIZE 32
new
    str[STR_SIZE],
    val;
sscanf("hello\\ there 27", "s[" #STR_SIZE "]i", str, val);
[/pawn]

So when you change your string size you don't need to change your specifiers.

    [LIST][*][b][color=red][u][color=green]Arrays[/color][/u][/color][/b][/LIST]

One of the advanced new specifiers is `a`, which creates an array, obviously. The syntax is similar to that of strings and, as you will see later, the delimiter code:

[pawn]
new
    arr[5];
sscanf("1 2 3 4 5", "a<i>[5]", arr);
[/pawn]

The `a` specifier is immediately followed by a single type enclosed in angle brackets - this type can be any of the basic types listed above. It is the followed, as with strings now, by an array size. The code above will put the numbers 1 to 5 into the 5 indexes of the `arr` array variable.

Arrays can now also be combined with strings (see below), specifying the string size in the array type:

[code]
a<s[10]>[12]
[/code]

This will produce an array of 12 strings, each up to 10 characters long (9 + NULL). Optional string arrays still follow the optional array syntax:

[code]
A<s[10]>(hello)[12]
[/code]

However, unlike numbers you can't specify a progression and have it fill up. This code:

[code]
A<i>(0, 1)[4]
[/code]

Will by default produce:

[code]
0, 1, 2, 3
[/code]

However, this code:

[code]
A<s[10]>(hi, there)[4]
[/code]

Will by default produce:

[code]
"hi, there", "hi, there", "hi, there", "hi, there"
[/code]

As normal, you can add brackets in to the default string value with `\)`:

[code]
A<s[10]>(hi (code\))[4]
[/code]

It should also be noted that there is NO length checking on default strings. If you do:

[code]
A<s[10]>(This is longer than 10 characters)[4]
[/code]

You will probably just corrupt the PAWN stack. The length checking is to ensure no users enter malicious data; however, in this case it is up to the scripter to ensure that the data is correct as they are the only one affecting it and shouldn't be trying to crash their own server. Interestingly, arrays of strings actually also work with jagged arrays and arrays that have been shuffled by Slice's quicksort function (this isn't a side-effect, I specifically wrote them to do so).

    [LIST][*][b][color=red][u][color=green]Enums[/color][/u][/color][/b][/LIST]

This is possibly the most powerful addition to sscanf ever. This gives you the ability to define the structure of an enum within your specifier string and read any data straight into it. The format takes after that of arrays, but with more types - and you can include strings in enums (but not other enums or arrays):

[pawn]
enum
    E_DATA
{
    E_DATA_C,
    Float:E_DATA_X,
    E_DATA_NAME[32],
    E_DATA_Z
}

main
{
    new
        var[E_DATA];
    sscanf("1 12.0 Bob c", "e<ifs[32]c>", var);
}
[/pawn]

Now I'll be impressed if you can read that code straight off, so I'll explain it slowly:

[pawn]
e - Start of the `enum` type
< - Starts the specification of the structure of the enum
i - An integer, corresponds with E_DATA_C
f - A float, corresponds with E_DATA_X
s[32] - A 32 cell string, corresponds with E_DATA_NAME
c - A character, corresponds with E_DATA_Z
> - End of the enum specification
[/pawn]

Note that an enum doesn't require a size like arrays and strings - it's size is determined by the number and size of the types. Most, but not all, specifiers can be used inside enums (notably arrays and other enums can't be).

    [LIST][*][b][color=red][u][color=green]Quiet[/color][/u][/color][/b][/LIST]

The two new specifiers `{` and `}` are used for what are known as `quiet` strings. These are strings which are read and checked, but not saved. For example:

[pawn]
sscanf("42 -100", "{i}i", var);
[/pawn]

Clearly there are two numbers and two `i`, but only one return variable. This is because the first `i` is quiet so is not saved, but affects the return value. The code above makes `var` `-100`. The code below will fail in an if check:

[pawn]
sscanf("hi -100", "{i}i", var);
[/pawn]

Although the first integer is not saved it is still read - and `hi` is not an integer. Quiet zones can be as long as you like, even for the whole string if you only want to check values are right, not save them:

[pawn]
sscanf("1 2 3", "i{ii}", var);
sscanf("1 2 3", "{iii}");
sscanf("1 2 3", "i{a<i>[2]}", var);
[/pawn]

You can also embed quiet sections inside enum specifications:

[pawn]
sscanf("1 12.0 Bob 42 INFINITY c", "e<ifs[32]{ig}c>", var);
[/pawn]

Quiet sections cannot contain other quiet sections, however they can include enums which contain quiet sections.

    [LIST][*][b][color=red][u][color=green]Searches[/color][/u][/color][/b][/LIST]

Searches were in the last version of sscanf too, but I'm explaining them again anyway. Strings enclosed in single quotes (') are scanned for in the main string and the position moved on. Note that to search for a single quote you escape it as above using `\\`:

[pawn]
sscanf("10 11 woo 12", "i'woo'i", var0, var1);
[/pawn]

Gives:

[code]
10
12
[/code]

You could achieve the same effect with:

[pawn]
sscanf("10 11 woo 12", "i{is[1000]}i", var0, var1);
[/pawn]

But that wouldn't check that the string was `woo`. Also note the use of `1000` for the string size. Quiet strings must still have a length, but as they aren't saved anywhere you can make this number as large as you like to cover any eventuality. Enum specifications can include search strings.

    [LIST][*][b][color=red][u][color=green]Enums[/color][/u][/color][/b][/LIST]

This is a feature similar to quiet sections, which allows you to skip overwriting certain parts of an enum:

[code]
e<ii-i-ii>
[/code]

Here the `-` is a `minus`, and tells sscanf that there is an enum element there, but not to do anything, so if you had:

[pawn]
enum E
{
    E_A,
    E_B,
    E_C,
    E_D,
    E_E
}
[/pawn]

And you only wanted to update the first two and the last fields and leave all others untouched you could use that specifier above. This way sscanf knows how to skip over the memory, and how much memory to skip. Note that this doesn't read anything, so you could also combine this with quiet sections:

[code]
e<ii-i-i{ii}i>
[/code]

That will read two values and save them, skip over two memory locations, read two values and NOT save them, then read and save a last value. In this way you can have written down all the values for every slot in the enum, but have only used 3 of them. Note that this is the same with `E` - if you do:

[code]
E<ii-i-ii>
[/code]

You should ONLY specify THREE defaults, not all five:

[code]
E<ii-i-ii>(11, 22, 55)
[/code]

    [LIST][*][b][color=red][u][color=green]Delimiters[/color][/u][/color][/b][/LIST]

The previous version of sscanf had `p` to change the symbol used to separate tokens. This specifier still exists but it has been formalised to match the array and enum syntax. What was previously:

[pawn]
sscanf("1,2,3", "p,iii", var0, var1, var2);
[/pawn]

Is now:

[pawn]
sscanf("1,2,3", "p<,>iii", var0, var1, var2);
[/pawn]

The old version will still work, but it will give a warning. Enum specifications can include delimiters, and is the only time `<>`s are contained in other `<>`s:

[pawn]
sscanf("1 12.0 Bob,c", "e<ifp<,>s[32]c>", var);
[/pawn]

Note that the delimiter will remain in effect after the enum is complete. You can even use `>` as a specifier by doing `p<\>>` (or the older `p>`).

When used with strings, the collection behaviour is overruled. Most specifiers are still space delimited, so for example this will work:

[pawn]
sscanf("1 2 3", "p<;>iii", var0, var1, var2);
[/pawn]

Despite the fact that there are no `;`s. However, strings will ONLY use the specified delimiters, so:

[pawn]
sscanf("hello 1", "p<->s[32]i", str, var);
[/pawn]

Will NOT work - the variable `str` will contain `hello 1`. On the other hand, the example from earlier, slightly modified:

[pawn]
sscanf("hello there>27", "p<>>s[32]i", str, var);
[/pawn]

WILL work and will give an output of:

[code]
hello there
27
[/code]

You can now have optional delimiters using `P` (upper case `p` to match other `optional` specifiers). These are optional in the sense that you specify multiple delimiters and any one of them can be used to end the next symbol:

[pawn]
sscanf("(4, 5, 6, 7)", "P<(),>{s[2]}iiii", a, b, c, d);
[/pawn]

This uses a `quiet section` to ignore anything before the first `(`, and then uses multiple delimiters to end all the text. Example:

[pawn]
sscanf("42, 43; 44@", "P<,;@>a<i>[3]", arr);
[/pawn]

    [LIST][*][b][color=red][u][color=green]Optional specifiers[/color][/u][/color][/b][/LIST]

EVERY format specifier (that is, everything except `''`, `{}` and `p`) now has an optional equivalent - this is just their letter capitalised. In addition to optional specifiers, there are also now default values:

[pawn]
sscanf("", "I(12)", var);
[/pawn]

The `()`s (round brackets) contain the default value for the optional integer and, as the main string has no data, the value of `var` becomes `12`. Default values come before array sizes and after specifications, so an optional array would look like:

[pawn]
sscanf("1 2", "A<i>(3)[4]", arr);
[/pawn]

Note that the size of the array is `4` and the default value is `3`. There are also two values which are defined, so the final value of `arr` is:

[code]
1, 2, 3, 3
[/code]

Array default values are clever, the final value of:

[pawn]
sscanf("", "A<i>(3,6)[4]", arr);
[/pawn]

Will be:

[code]
3, 6, 9, 12
[/code]

The difference between `3` and `6` is `3`, so the values increase by that every index. Note that it is not very clever, so:

[pawn]
sscanf("", "A<i>(1,2,2)[4]", arr);
[/pawn]

Will produce:

[code]
1, 2, 2, 2
[/code]

The difference between `2` and `2` (the last 2 numbers in the default) is 0, so there will be no further increase. For `l` (logical) arrays, the value is always the same as the last value, as it is with `g` if the last value is one of the special values (INFINITY, NEG_INFINITY (same as -INFINITY), NAN or NAN_E). Note that:

[pawn]
sscanf("", "a<I>(1,2,2)[4]", arr);
[/pawn]

Is invalid syntax, the `A` must be the capital part.

Enums can also be optional:

[pawn]
sscanf("4", "E<ifs[32]c>(1, 12.0, Bob, c)", var);
[/pawn]

In that code all values except `4` will be default. Also, again, you can escape commas with `\\` in default enum strings. Some final examples:

[pawn]
sscanf("1", "I(2)I(3)I(4)", var0, var1, var2);
sscanf("", "O(045)H(0xF4)B(0b0100)U(Y_Less)", octnum, hexnum, binnum, user);
sscanf("0xFF", "N(0b101)");
[/pawn]

That last example is of a specifier not too well described yet - the `number` specifier, which will work out the format of the number from the leading characters (0x, 0b, 0 or nothing). Also note that the second example has changed - see the next section.

    [LIST][*][b][color=red][u][color=green]Users[/color][/u][/color][/b][/LIST]

The `u`, `q`, and `r` specifiers search for a user by name or ID. The method of this search has changed in the latest versions of `sscanf`.

Additionally `U`, `Q`, and `R` used to take a name or ID as their default value - this has since been changed to JUST a number, and sscanf will not try and determine if this number is online:

Previous:

[pawn]
sscanf(params, "U(Y_Less)", id);
if (id == INVALID_PLAYER_ID)
{
    // Y_Less or the entered player is not connected.
}
[/pawn]

New:

[pawn]
sscanf(params, "U(-1)", id);
if (id == -1)
{
    // No player was entered.
}
else if (id == INVALID_PLAYER_ID)
    // Entered player is not connected.
}
[/pawn]

See the section on options for more details.

Users can now optionally return an ARRAY of users instead of just one. This array is just a list of matched IDs, followed by `INVALID_PLAYER_ID`. Given the following players:

[code]
0) Y_Less
1) [CLAN]Y_Less
2) Jake
3) Alex
4) Hass
[/code]

This code:

[pawn]
new ids[3], i;
if (sscanf("Le", "?<MATCH_NAME_PARTIAL=1>u[3]", ids)) printf("Error in input");
for (i = 0; ids[i] != INVALID_PLAYER_ID; ++i)
{
    if (ids[i] == cellmin)
    {
        printf("Too many matches");
        break;
    }
    printf("id = %d", ids[i]);
}
if (i == 0) printf("No matching players found.");
[/pawn]

Will output:

[code]
id = 0
id = 1
Too many matches
[/code]

Searching `Les` instead will give:

[code]
id = 0
id = 1
[/code]

And searching without `MATCH_NAME_PARTIAL` will give:

[code]
No matching players found.
[/code]

Basically, if an array of size `N` is passed, this code will return the first N-1 results. If there are less than `N` players whose name matches the given name then that many players will be returned and the next slot will be `INVALID_PLAYER_ID` to indicate the end of the list. On the other hand if there are MORE than `N - 1` players whose name matches the given pattern, then the last slot will be `cellmin` to indicate this fact.

When combined with `U` and returning the default, the first slot is always exactly the default value (even if that's not a valid connected player) and the next slot is always `INVALID_PLAYER_ID`.

Note also that user arrays can't be combined with normal arrays or enums, but normal single-return user specifiers still can be.

    [LIST][*][b][color=red][u][color=green]Custom (kustom) specifiers[/color][/u][/color][/b][/LIST]

The latest version of sscanf adds a new `k` specifier to allow you to define your own specifers in PAWN:

[pawn]
SSCANF:playerstate(string[])
{
    if ('0' <= string[0] <= '9')
    {
        new
            ret = strval(string);
        if (0 <= ret <= 9)
        {
            return ret;
        }
    }
    else if (!strcmp(string, "PLAYER_STATE_NONE")) return 0;
    else if (!strcmp(string, "PLAYER_STATE_ONFOOT")) return 1;
    else if (!strcmp(string, "PLAYER_STATE_DRIVER")) return 2;
    else if (!strcmp(string, "PLAYER_STATE_PASSENGER")) return 3;
    else if (!strcmp(string, "PLAYER_STATE_WASTED")) return 7;
    else if (!strcmp(string, "PLAYER_STATE_SPAWNED")) return 8;
    else if (!strcmp(string, "PLAYER_STATE_SPECTATING")) return 9;
}
[/pawn]

The code above, when added to the top level of your mode, will add the `playerstate` specifier, allowing you to do:

[pawn]
sscanf(params, "uk<playerstate>", playerid, state);
[/pawn]

This system supports optional custom specifiers with no additional PAWN code:

[pawn]
sscanf(params, "uK<playerstate>(PLAYER_STATE_NONE)", playerid, state);
[/pawn]

The new version of `sscanf2.inc` includes functions for `k<weapon>` and `k<vehicle>` allowing you to enter either the ID or name and get the ID back, but both are VERY basic at the moment and I expect other people will improve on them.

Note that custom specifiers are not supported in either arrays or enumerations.

Note also that custom specifiers always take a string input and always return a number, but this can be a Float, bool, or any other single cell tag type.

The optional kustom specifier `K` takes a default value that is NOT (as of sscanf 2.8) parsed by the given callback:

[code]
K<vehicle>(999)
[/code]

`999` is NOT a valid vehicle model, but if no other value is supplied then 999 will be returned, allowing you to differentiate between the user entering an invalid vehicle and not entering anything at all.

Also as of sscanf 2.8, `k` can be used in both arrays and enums.

## Options[/color][/u][/color][/b][/size]

The latest version of sscanf introduces several options that can be used to customise the way in which sscanf operates. There are two ways of setting these options - globally and locally:

[pawn]
SSCANF_Option(SSCANF_QUIET, 1);
[/pawn]

This sets the `SSCANF_QUIET` option globally. Every time `sscanf` is called the option (see below) will be in effect. Note that the use of:

[code]
SSCANF_QUIET
[/code]

Instead of a string as:

[code]
SSCANF_QUIET
[/code]

Is entirely valid here - all the options are defined in the sscanf2 include already.

Alternatively you can use `?` to specify an option locally - i.e. only for the current sscanf call:

[pawn]
sscanf(params, "si", str, num);
sscanf(params, "?<SSCANF_QUIET=1>si", str, num);
sscanf(params, "si", str, num);
[/pawn]

Obviously `s` without a length is deprecated, and the first and last `sscanf` calls will give a warning in the console, but the second one won't as for just that one call prints have been disabled. The following code disables prints globally then enables them locally:

[pawn]
SSCANF_Option(SSCANF_QUIET, 1);
sscanf(params, "si", str, num);
sscanf(params, "?<SSCANF_QUIET=0>si", str, num);
sscanf(params, "si", str, num);
[/pawn]

Note that disabling prints is a VERY bad idea when developing code as you open yourself up to unreported buffer overflows when no length is specified on strings less than 32 cells (the default length).

To specify multiple options requires multiple calls:

[pawn]
SSCANF_Option(SSCANF_QUIET, 1);
SSCANF_Option(MATCH_NAME_PARTIAL, 0);
sscanf(params, "?<SSCANF_QUIET=1>?<MATCH_NAME_PARTIAL=0>s[10]i", str, num);
[/pawn]

The options are:

    [LIST][*][b][color=red][u][color=green]OLD_DEFAULT_NAME:[/color][/u][/color][/b][/LIST]

    The behaviour of `U`, `Q`, and `R` have been changed to take any number as a default, instead of a connected player. Setting `OLD_DEFAULT_NAME` to `1` will revert to the old version.

    [LIST][*][b][color=red][u][color=green]MATCH_NAME_PARTIAL:[/color][/u][/color][/b][/LIST]

    Currently sscanf will search for players by name, and will ALWAYS search for player whose name STARTS with the specified string. If you have, say `[CLAN]Y_Less` connected and someone types `Y_Less`, sscanf will not find `[CLAN]Y_Less` because there name doesn't start with the specified name. This option, when set to 1, will search ANYWHERE in the player's name for the given string.

    [LIST][*][b][color=red][u][color=green]CELLMIN_ON_MATCHES:[/color][/u][/color][/b][/LIST]

    Whatever the value of `MATCH_NAME_PARTIAL`, the first found player will always be returned, so if you do a search for `_` on an RP server, you could get almost anyone. To detect this case, if more than one player will match the specified string then sscanf will return an ID of `cellmin` instead. This can be combined with `U` for a lot more power:

    [pawn]
    sscanf(params, "?<CELLMIN_ON_MATCHES=1>U(-1)", id);
    if (id == -1)
    {
        // No player was entered.
    }
    else if (id == cellmin)
    {
        // Multiple matches found
    }
    else if (id == INVALID_PLAYER_ID)
    {
        // Entered player is not connected.
    }
    else
    {
        // Found just one player.
    }
    [/pawn]

    [LIST][*][b][color=red][u][color=green]SSCANF_QUIET:[/color][/u][/color][/b][/LIST]

    Don't print any errors to the console. REALLY not recommended unless you KNOW your code is stable and in production.

    [LIST][*][b][color=red][u][color=green]OLD_DEFAULT_KUSTOM:[/color][/u][/color][/b][/LIST]

    As with `U`, `K` used to require a valid identifier as the default and would parse it using the specified callback, so this would NOT work:

    [code]
    K<vehicle>(Veyron)
    [/code]

    Because that is not a valid vehicle name in GTA. The new version now JUST takes a number and returns that regardless:

    [code
    K<vehicle>(9999)
    [/code]

    This setting reverts to the old behaviour.

## All specifiers[/color][/u][/color][/b][/size]

For quick reference, here is a list of ALL the specifiers and their use:

[code]
[b][u]Format[/u][/b]					[b][u]Use[/u][/b]
L(true/false)				Optional logical truthity
l					Logical truthity
K<callback>(any format number)	        Optional custom operator
k<callback>				Custom operator
B(binary)				Optional binary number
b					Binary number
N(any format number)			Optional number
n					Number
C(character)				Optional character
c					Character
I(integer)				Optional integer
i					Integer
D(integer)				Optional integer
d					Integer
H(hex value)				Optional hex number
h					Hex number
O(octal value)				Optional octal value
o					Octal value
F(float)				Optional floating point number
f					Floating point number
G(float/INFINITY/-INFINITY/NAN/NAN_E)	Optional float with IEEE definitions
g					Float with IEEE definitions
{					Open quiet section
}					Close quiet section
P<delimiters>				Multiple delimiters change
p<delimiter>				Delimiter change
Z(string)[length]			Invalid optional string
z(string)[length]			Deprecated optional string
S(string)[length]			Optional string
s[length]				String
U(any format number)			Optional user (bot/player)
u					User (bot/player)
Q(any format number)			Optional bot (bot)
q					Bot (bot)
R(any format number)			Optional player (player)
r					Player (player)
A<type>(default)[length]		Optional array of given type
a<type>[length]				Array of given type
E<specification>(default)		Optional enumeration of given layout
e<specification>			Enumeration of given layout
'string'				Search string
%					Deprecated optional specifier prefix
?					Local options specifier
[/code]

## `extract`[/color][/u][/color][/b][/size]

I've written some (extendable) macros so you can do:

[pawn]
extract params -> new a, string:b[32], Float:c; else
{
    return SendClientMessage(playerid, COLOUR_RED, "FAIL!");
}
[/pawn]

This will compile as:

[pawn]
new a, string:b[32], Float:c;
if (unformat(params, "is[32]f", a, b, c))
{
    return SendClientMessage(playerid, COLOUR_RED, "FAIL!");
}
[/pawn]

Note that `unformat` is the same as `sscanf`, also note that the `SendClientMessage` part is optional:

[pawn]
extract params -> new a, string:b[32], Float:c;
[/pawn]

Will simply compile as:

[pawn]
new a, string:b[32], Float:c;
unformat(params, "is[32]f", a, b, c);
[/pawn]

Basically it just simplifies sscanf a little bit (IMHO). I like new operators and syntax, hence this, examples:

[pawn]
// An int and a float.
extract params -> new a, Float:b;
// An int and an OPTIONAL float.
extract params -> new a, Float:b = 7.0;
// An int and a string.
extract params -> new a, string:s[32];
// An int and a playerid.
extract params -> new a, player:b;
[/pawn]

As I say, the syntax is extendable, so to add hex numbers you would do:

[pawn]
#define hex_EXTRO:%0##%1,%2|||%3=%9|||%4,%5) EXTRY:%0##%1H"("#%9")"#,%2,%3|||%4|||%5)
#define hex_EXTRN:%0##%1,%2|||%3|||%4,%5) EXTRY:%0##%1h,%2,%3|||%4|||%5)
#define hex_EXTRW:%0##%1,%2|||%3[%7]|||%4,%5) EXTRY:%0##%1a<h>[%7],%2,%3|||%4|||%5)
[/pawn]

That will add the tag `hex` to the system. Yes, the lines look complicated (because they are), but the ONLY things you need to change are the name before the underscore and the letter near the middle (`H`, `h` and `a<h>` in the examples above for `optional`, `required` and `required array` (no optional arrays yet besides strings)).

New examples (with `hex` added):

[pawn]
// A hex number and a player.
extract params -> new hex:a, player:b;
// 32 numbers then 32 players.
extract params -> new a[32], player:b[32];
// 11 floats, an optional string, then an optional hex number.
extract params -> new Float:f[11], string:s[12] = "optional", hex:end = 0xFF;
[/pawn]

The code is actually surprisingly simple (I developed another new technique to simplify my `tag` macros and it paid off big style here). By default `Float`, `string`, `player` and `_` (i.e. no tag) are supported, and their individual letter definitions take up the majority of the code as demonstrated with the `hex` addition above. Note that `string:` is now used extensively in my code to differentiate from tagless arrays in cases like this, it is removed automatically but `player:` and `hex:` are not so you may wish to add:

[pawn]
#define player:
#define hex:
[/pawn]

To avoid tag mismatch warnings (to remove them AFTER the compiler has used them to determine the correct specifier).

The very first example had an `else`, this will turn:

[pawn]
unformat(params, "ii", a, b);
[/pawn]

In to:

[pawn]
if (unformat(params, "ii", a, b))
[/pawn]

You MUST put the `else` on the same line as `extract` for it to be detected, but then you can use normal single or multi-line statements. This is to cover common command use cases, you can even leave things on the same line:

[pawn]
else return SendClientMessage(playerid, 0xFF0000AA, "Usage: /cmd <whatever>");
[/pawn]

There is now the ability to split by things other than space (i.e. adds `P<?>` to the syntax - updated from using `p` to `P`):

[pawn]
extract params<|> -> new a, string:b[32], Float:c;
[/pawn]

Will simply compile as:

[pawn]
new a, string:b[32], Float:c;
unformat(params, "P<|>is[32]f", a, b, c);
[/pawn]

Note that for technical reasons you can use `<->` (because it looks like the arrow after the `extract` keyword). You also can't use `<;>`, `<,>`, or `<)>` because of a bug with `#`, but you can use any other character (most notably `<|>`, as is popular with SQL scripts). I'm thinking of adding enums and existing variables (currently you HAVE to declare new variables), but not right now.

## Errors/Warnings[/color][/u][/color][/b][/size]

    [LIST][*][b][color=red][u][color=green]MSVRC100.dll not found[/color][/u][/color][/b][/LIST]

If you get this error, DO NOT just download the dll from a random website. This is part of the `Microsoft Visual Studio Redistributable Package`. This is required for many programs, but they often come with it. Download it here:

[url]http://www.microsoft.com/download/en...s.aspx?id=5555[/url]

    [LIST][*][b][color=red][u][color=green]sscanf error: System not initialised[/color][/u][/color][/b][/LIST]

If you get this error, you need to make sure that you have recompiled ALL your scripts using the LATEST version of `sscanf2.inc`. Older versions didn't really require this as they only had two natives - `sscanf` and `unformat`, the new version has some other functions - you don't need to worry about them, but you must use `sscanf2.inc` so that they are correctly called. If you think you have done this and STILL get the error then try again - make sure you are using the correct version of PAWNO for example.

    [LIST][*][b][color=red][u][color=green]sscanf warning: String buffer overflow.[/color][/u][/color][/b][/LIST]

This error comes up when people try and put too much data in to a string. For example:

[pawn]
new str[10];
sscanf("Hello there, how are you?", "s[10]", str);
[/pawn]

That code will try and put the string `Hello there, how are you?` in to the variable called `str`. However, `str` is only 10 cells big and can thus only hold the string `Hello ther` (with a NULL terminator). In this case, the rest of the data is ignored - which could be good or bad:

[pawn]
new str[10], num;
sscanf("Hello there you|42", "p<|>s[10]i", str, num);
[/pawn]

In this case `num` is still correctly set to `42`, but the warning is given for lost data (`e you`).

Currently there is nothing you can do about this from a programming side (you can't even detect it - that is a problem I intend to address), as long as you specify how much data a user should enter this will simply discard the excess, or make the destination variable large enough to handle all cases.

[LIST][*][b][color=red][u][color=green]sscanf warning: Optional types invalid in array specifiers, consider using 'A'.[/color][/u][/color][/b][/LIST]

A specifier such as:

[code]
a<I(5)>[10]
[/code]

Has been written - here indicating an array of optional integers all with the default value `5`. Instead you should use:

[code]
A<i>(5)[10]
[/code]

This is an optional array of integers all with the default value `5`, the advantage of this is that arrays can have multiple defaults:

[code]
A<i>(5, 6)[10]
[/code]

That will set the array to `5, 6, 7, 8, 9, 10, 11, 12, 13, 14` by default, incrementing by the found difference each time.

[LIST][*][b][color=red][u][color=green]sscanf warning: Optional types invalid in enum specifiers, consider using 'E'.[/color][/u][/color][/b][/LIST]

Similar to the previous warning, A specifier such as:

[code]
e<I(5)f>
[/code]

Is invalid, instead use:

[code]
E<if>(42, 11.0)
[/code]

This forces ALL the parts of an enum to be optional - anything less is not possible.

    [LIST][*][b][color=red][u][color=green]sscanf error: Multi-dimensional arrays are not supported.[/color][/u][/color][/b][/LIST]

This is not allowed:

[pawn]
sscanf(params, "a<a<i>[5]>[10]", arr);
[/pawn]

A work-around can be done using:

[pawn]
sscanf(params, "a<i>[50]", arr[0]);
[/pawn]

That will correctly set up the pointers for the system.

    [LIST][*][b][color=red][u][color=green]sscanf error: Search strings are not supported in arrays.[/color][/u][/color][/b][/LIST]

This is not allowed (see the section on search strings):

[code]
a<'hello'i>[10]
[/code]

    [LIST][*][b][color=red][u][color=green]sscanf error: Delimiters are not supported in arrays.[/color][/u][/color][/b][/LIST]

This is not allowed:

[code]
a<p<,>i>[10]
[/code]

Instead use:

[code]
p<,>a<i>[10]
[/code]

    [LIST][*][b][color=red][u][color=green]sscanf error: Quiet sections are not supported in arrays.[/color][/u][/color][/b][/LIST]


This is not allowed:

[code]
a<{i}>[10]
[/code]

Instead use:

[code]
{a<i>[10]}
[/code]

    [LIST][*][b][color=red][u][color=green]sscanf error: Unknown format specifier '?'.[/color][/u][/color][/b][/LIST]

The given specifier is not known (this post contains a full list of all the specifiers near the bottom).

    [LIST][*][b][color=red][u][color=green]sscanf warning: Empty default values.[/color][/u][/color][/b][/LIST]

An optional specifier has been set as (for example):

[code]
I()
[/code]

Instead of:

[code]
I(42)
[/code]

This does not apply to strings as they can be legitimately empty.

    [LIST][*][b][color=red][u][color=green]sscanf warning: Unclosed default value.[/color][/u][/color][/b][/LIST]

You have a default value on an optional specifier that looks like:

[code]
I(42
[/code]

Instead of:

[code]
I(42)
[/code]

    [LIST][*][b][color=red][u][color=green]sscanf warning: No default value found.[/color][/u][/color][/b][/LIST]

You have no default value on an optional specifier:

[code]
I
[/code]

Instead of:

[code]
I(42)
[/code]

[LIST][*][b][color=red][u][color=green]sscanf warning: Unclosed specifier parameter, assuming '<', consider using something like p<<>.[/color][/u][/color][/b][/LIST]

A custom delimiter of:

[code]
p<
[/code]

Was found with no matching `>` after one character. In this case the system assumes you are using the old (deprecated) style of delimiters and sets it to just `<`. Instead use:

[code]
p<,>
[/code]

Or, if you really do want a delimiter of `<` then use:

[code]
p<<>
[/code]

Note that this does not need to be escaped; however, a delimiter of `>` does:

[code]
p<\>>
[/code]

The `\` may also need to be escaped when writing actual PAWN strings, leading to:

[code]
p<\\>>
[/code]

This also applies to array types (`a<` vs `a<i>`), note that this will result in an invalid array type.

[LIST][*][b][color=red][u][color=green]sscanf warning: Unenclosed specifier parameters are deprecated, consider using something like p<<>.[/color][/u][/color][/b][/LIST]

You are using the old style:

[code]
p,
[/code]

Instead of:

[code]
p<,>
[/code]

This also applies to array types (`ai` vs `a<i>`).

[LIST][*][b][color=red][u][color=green]sscanf warning: No specified parameter found.[/color][/u][/color][/b][/LIST]

The format specifier just ends with:

[code]
p
[/code]

This also applies to array types (`a` vs `a<i>`).

[LIST][*][b][color=red][u][color=green]sscanf warning: Missing string length end.[/color][/u][/color][/b][/LIST]
[LIST][*][b][color=red][u][color=green]sscanf warning: Missing length end.[/color][/u][/color][/b][/LIST]

A string has been written as:

[code]
s[10
[/code]

Instead of:

[code]
s[10]
[/code]

I.e. the length has not been closed.

[LIST][*][b][color=red][u][color=green]sscanf warning: Arrays without a length are deprecated, please add a destination size.[/color][/u][/color][/b][/LIST]

A string has been written as:

[code]
s
[/code]

Instead of:

[code]
s[10]
[/code]

I.e. the length has not been included.

    [LIST][*][b][color=red][u][color=green]sscanf error: Invalid data length.[/color][/u][/color][/b][/LIST]

An invalid array or string size has been specified (0, negative, or not a number).

    [LIST][*][b][color=red][u][color=green]sscanf error: Invalid character in data length.[/color][/u][/color][/b][/LIST]

A string or array has been given a length that is not a number.

    [LIST][*][b][color=red][u][color=green]sscanf warning: Strings without a length are deprecated, please add a destination size.[/color][/u][/color][/b][/LIST]

In the old system, strings were not required to have lengths but this introduced security problems with overflows. Now you must add a length or get the default of `32`.

    [LIST][*][b][color=red][u][color=green]sscanf error: String/array must include a length, please add a destination size.[/color][/u][/color][/b][/LIST]

Arrays are newer than strings, so never had an implementation not requiring a length, so there is no compatability problems in REQUIRING a length to be given.

    [LIST][*][b][color=red][u][color=green]sscanf warning: Can't have nestled quiet sections.[/color][/u][/color][/b][/LIST]

You have tried writing something like this:

[code]
{i{x}}
[/code]

This has a quiet section (`{}`) inside another one, which makes no sense.

    [LIST][*][b][color=red][u][color=green]sscanf warning: Not in a quiet section.[/color][/u][/color][/b][/LIST]

`}` was found with no corresponding `{`:

[code]
i}
[/code]

    [LIST][*][b][color=red][u][color=green]sscanf warning: Can't remove quiet in enum.[/color][/u][/color][/b][/LIST]

This is caused by specifiers such as:

[code]
{fe<i}x>
[/code]

Where the quiet section is started before the enum, but finishes part way through it rather than after it. This can be emulated by:

[code]
{f}e<{i}x>
[/code]

    [LIST][*][b][color=red][u][color=green]sscanf error: Arrays are not supported in enums.[/color][/u][/color][/b][/LIST]


Basically, you can't do:

[code]
e<fa<i>[5]f>
[/code]

You can, however, still do:

[code]
e<fiiiiif>
[/code]

This is a little more awkward, but is actually more technically correct given how enums are compiled.

    [LIST][*][b][color=red][u][color=green]sscanf warning: Unclosed string literal.[/color][/u][/color][/b][/LIST]

A specifier starts a string with `'`, but doesn't close it:

[code]
i'hello
[/code]

[LIST][*][b][color=red][u][color=green]sscanf warning: sscanf specifiers do not require '%' before them.[/color][/u][/color][/b][/LIST]

`format` uses code such as `%d`, sscanf only needs `d`, and confusingly the C equivalent function (also called `sscanf`) DOES require `%`. Sorry.

[LIST][*][b][color=red][u][color=green]sscanf error: Insufficient default values.[/color][/u][/color][/b][/LIST]

Default values for arrays can be partially specified and the remainder will be inferred from the pattern of the last two:

[code]
A<i>(0, 1)[10]
[/code]

That specifier will default to the numbers `0` to `9`. However, because enums have a mixture of types, all the default values for `E` must ALWAYS be specified:

[code]
E<iiff>(0, 1, 0.0, 1.0)
[/code]

This will not do:

[code]
E<iiff>(0, 1)
[/code]

[LIST][*][b][color=red][u][color=green]sscanf error: Options are not supported in enums.[/color][/u][/color][/b][/LIST]

[LIST][*][b][color=red][u][color=green]sscanf error: Options are not supported in arrays.[/color][/u][/color][/b][/LIST]

The `?` specifier for local options must appear outside any other specifier.

    [LIST][*][b][color=red][u][color=green]sscanf error: No option value.[/color][/u][/color][/b][/LIST]

An option was specified with no value:

[code]
?<OLD_DEFAULT_NAME>
[/code]

    [LIST][*][b][color=red][u][color=green]sscanf error: Unknown option name.[/color][/u][/color][/b][/LIST]

The given option was not recognised. Check spelling and case:

[code]
?<NOT_A_VALID_NAME=1>
[/code]

[LIST][*][b][color=red][u][color=green]sscanf warning: Could not find function SSCANF:?.[/color][/u][/color][/b][/LIST]

A `k` specifier has been used, but the corresponding function could not be found. If you think it is there check the spelling matches exactly - including the case.

[LIST][*][b][color=red][u][color=green]sscanf error: SSCANF_Init has incorrect parameters.[/color][/u][/color][/b][/LIST]
[LIST][*][b][color=red][u][color=green]sscanf error: SSCANF_Join has incorrect parameters.[/color][/u][/color][/b][/LIST]
[LIST][*][b][color=red][u][color=green]sscanf error: SSCANF_Leave has incorrect parameters.[/color][/u][/color][/b][/LIST]
[LIST][*][b][color=red][u][color=green]sscanf error: SSCANF_SetPlayerName has incorrect parameters.[/color][/u][/color][/b][/LIST]

You edited something in the sscanf2 include - undo it or redownload it.

## Changelog

[b][color=red][u][color=green]sscanf 2.8.2 - 18/04/2015[/color][/u][/color][/b]
- Fixed a bug where `u` wasn't working correctly after a server restart.[/QUOTE]


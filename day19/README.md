```python
import re
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt
```

# Day 19: Monster Messages

Reference: https://adventofcode.com/2020/day/19

## Part 1
You land in an airport surrounded by dense forest. As you walk to your high-speed train, the Elves at the Mythical Information Bureau contact you again. They think their satellite has collected an image of a sea monster! Unfortunately, the connection to the satellite is having problems, and many of the messages sent back from the satellite have been corrupted.

They sent you a list of the rules valid messages should obey and a list of received messages they've collected so far (your puzzle input).

The rules for valid messages (the top part of your puzzle input) are numbered and build upon each other. For example:
```
0: 1 2
1: "a"
2: 1 3 | 3 1
3: "b"
```
Some rules, like 3: "b", simply match a single character (in this case, b).

The remaining rules list the sub-rules that must be followed; for example, the rule 0: 1 2 means that to match rule 0, the text being checked must match rule 1, and the text after the part that matched rule 1 must then match rule 2.

Some of the rules have multiple lists of sub-rules separated by a pipe (|). This means that at least one list of sub-rules must match. (The ones that match might be different each time the rule is encountered.) For example, the rule 2: 1 3 | 3 1 means that to match rule 2, the text being checked must match rule 1 followed by rule 3 or it must match rule 3 followed by rule 1.

Fortunately, there are no loops in the rules, so the list of possible matches will be finite. Since rule 1 matches a and rule 3 matches b, rule 2 matches either ab or ba. Therefore, rule 0 matches aab or aba.

Here's a more interesting example:
```
0: 4 1 5
1: 2 3 | 3 2
2: 4 4 | 5 5
3: 4 5 | 5 4
4: "a"
5: "b"
```
Here, because rule 4 matches a and rule 5 matches b, rule 2 matches two letters that are the same (aa or bb), and rule 3 matches two letters that are different (ab or ba).

Since rule 1 matches rules 2 and 3 once each in either order, it must match two pairs of letters, one pair with matching letters and one pair with different letters. This leaves eight possibilities: aaab, aaba, bbab, bbba, abaa, abbb, baaa, or babb.

Rule 0, therefore, matches a (rule 4), then any of the eight options from rule 1, then b (rule 5): aaaabb, aaabab, abbabb, abbbab, aabaab, aabbbb, abaaab, or ababbb.

The received messages (the bottom part of your puzzle input) need to be checked against the rules so you can determine which are valid and which are corrupted. Including the rules and the messages together, this might look like:
```
0: 4 1 5
1: 2 3 | 3 2
2: 4 4 | 5 5
3: 4 5 | 5 4
4: "a"
5: "b"

ababbb
bababa
abbbab
aaabbb
aaaabbb
```
Your goal is to determine the number of messages that completely match rule 0. In the above example, ababbb and abbbab match, but bababa, aaabbb, and aaaabbb do not, producing the answer 2. The whole message must match all of rule 0; there can't be extra unmatched characters in the message. (For example, aaaabbb might appear to match rule 0 above, but it has an extra unmatched b on the end.)

How many messages completely match rule 0?


```python
rules_dict = dict()
dat = list()
mode = 'rules'
with open('satt_input.txt', 'r') as fid:
    for line in fid:
        line = line.strip()
        if not line:
            # Empty string, switch modes
            mode = 'data'
            continue
        if 'rules' == mode:
            n, s = line.split(': ')
            rules_dict[int(n)] = s.strip('"')
        elif 'data' == mode:
            dat.append(line)

# Convert rules to a list, so it can just be
# dereferenced by index of the list by sorting
# the rules dict and converting the values
rules_raw = list({k: rules_dict[k] for k in sorted(rules_dict)}.values())
```


```python
test_rules_raw1 = ['1 2',
              'a',
              '1 3 | 3 1',
              'b']
test_rules_raw2 = ['4 1 5',
              '2 3 | 3 2',
              '4 4 | 5 5',
              '4 5 | 5 4',
              'a',
              'b']
```

### Strategy

Work backwards.
Start with list items for a & b and go back and
replace all references to to them with actual a & b's
Then find any rules that don't have any numbers in them.
Go through and replace references to them with a & b's
Continue until all list item references have been resolved.


```python
# Regex for matching on digits will be ver helpful:
rdig = re.compile('\d+')

# So as not to stack up redundant levels of parenthesis,
# create a function to determine whether the parenthesis
# within a string are balanced or not:
def get_balanced(txt):
    # Check if supplied txt string has balanced
    # parens. Return True if so, False otherwise
    stack = list()
    for c in txt:
        if '(' == c:
            stack.append(c)
        elif ')' == c:
            if 1 > len(stack):
                return False
            else:
                stack.pop(0)
    if 0 < len(stack):
        return False
    else:
        return True

# This is our workhorse function which will go through
# and resolve rule references by replacing them with the
# rules they refer to:
def rep_rules(rules, skipset=set()):
    rules_deref = rules.copy()
    rules_deref2 = rules.copy()
    for i, rule in enumerate(rules_deref):
        if i not in skipset and not rdig.search(rule):
        #if not rdig.search(rule):
            #print("{}: {}".format(i, rule))
            # No references in this rule (and rule is not in skipset)
            # Find all rules that reference this rule
            # and replace those references with this string
            # Must match non-digit then digits followed by a non-digit so as
            # not to match part of a number
            ri = re.compile('(^|(?<=\D)){}(?=\D|$)'.format(i))
            for j, rule2 in enumerate(rules_deref2):
                if 1 < len(rule):
                    # Check if it's already contained in parens
                    if rule[0] != '(' and rule[-1] != ')':
                        # Rule is not encased in parenthesis, defintely add them
                        rule = '({})'.format(rule)
                    else:
                        # Assume outer parens exist... but are they top level
                        # parens or part of several paren expressions? Check
                        # by seeing if the substring that excludes the first
                        # and last characters are paren balanced. If they are,
                        # we don't need any more parens. If they aren't we do.
                        if not get_balanced(rule[1:-1]):
                            rule = '({})'.format(rule)
                newrule, nsubs = ri.subn(rule, rule2)
                if 0 < nsubs:
                    # Only if substitutions were made...
                    #print("newrule: ", newrule)
                    rules_deref2[j] = newrule
            skipset.add(i)
    #return rules_deref3, skipset
    return rules_deref2, skipset
```


```python
# For testing
#rules_deref = rules_raw.copy()
#skipset = set()
#rrs = [0, 8, 2, 130, 92, 19, 9]
#for j in range(5):
#    print("=-"*20)
#    rules_deref, skipset = rep_rules(rules_deref, skipset)
#    for i in rrs:
#        print("{}: {}".format(i, rules_deref[i].replace(' ', '')))
```


```python
# Repeatedly call rep_rules() on our ruleset until
# rule 0 is fully resolved:
#rules_deref = test_rules_raw2.copy()
rules_deref = rules_raw.copy()
skipset = set()
last_skipset = set()
#while len(skipset) < len(rules_deref):
keep_going = True
while(keep_going):
    rules_deref, skipset = rep_rules(rules_deref, skipset)
    r0 = rules_deref[0]
    keep_going = bool(rdig.search(r0))
    if last_skipset == skipset:
        print("Did not get any new matches")
        keep_going = False
    last_skipset = skipset.copy()
#print(rules_deref[0])
# Make regex including beginning (^) and end ($) assertions
# so patterns with extraneous text are not matched
r0 = '^' + rules_deref[0].replace(' ', '') + '$'
print("Our completely assembled regex is:")
print(r0)
```

    Our completely assembled regex is:
    ^((a(a(b(b(b(a(a|b)|ba)|a(bb))|a(a(ba|(a|b)b)|b(bb|aa)))|a(a((bb|aa)b|(ba|(a|b)b)a)|b(b(aa|ab)|a(ab|bb))))|b(a(b((ba|(a|b)b)a|(aa|b(a|b))b)|a((ba|aa)b|(bb|aa)a))|b(a(a(ba)|b(ba|aa))|b(a(ba|(a|b)b)|b(a(a|b)|ba)))))|b(b((((ab|bb)a|(ba)b)a|(a(ab)|b(ba|(a|b)b))b)b|((a(ba|aa)|b(aa|ab))b|((bb)a)a)a)|a(b(((aa|ab)a|((a|b)(a|b))b)a|(b(aa|ab)|a(ab|bb))b)|a(((a(a|b)|ba)b|(aa|b(a|b))a)b|(b(aa)|a(aa))a))))b|(b(a(a(b((bb)b|(ba|aa)a)|a((bb|a(a|b))a|(ab)b))|b(((aa)b|(bb|a(a|b))a)b|((ba|(a|b)b)a|(bb|ba)b)a))|b(a(b(a(aa)|b(bb|a(a|b)))|a((bb|a(a|b))(a|b)))|b(b(b(bb|a(a|b))|a(ab|bb))|a((bb|ba)(a|b)))))|a(b((a((bb)b|(ba|aa)a)|b(b(bb|a(a|b))|a(ab)))a|(a(b(bb)|a(aa|b(a|b)))|b((ab)a))b)|a((b((bb|a(a|b))b|(ba|(a|b)b)a)|a((ab)a|(aa|ab)b))a|(a(b(ba|aa)|a(aa))|b((ab)b|(ab)a))b)))a)(((a(a(b(b(b(a(a|b)|ba)|a(bb))|a(a(ba|(a|b)b)|b(bb|aa)))|a(a((bb|aa)b|(ba|(a|b)b)a)|b(b(aa|ab)|a(ab|bb))))|b(a(b((ba|(a|b)b)a|(aa|b(a|b))b)|a((ba|aa)b|(bb|aa)a))|b(a(a(ba)|b(ba|aa))|b(a(ba|(a|b)b)|b(a(a|b)|ba)))))|b(b((((ab|bb)a|(ba)b)a|(a(ab)|b(ba|(a|b)b))b)b|((a(ba|aa)|b(aa|ab))b|((bb)a)a)a)|a(b(((aa|ab)a|((a|b)(a|b))b)a|(b(aa|ab)|a(ab|bb))b)|a(((a(a|b)|ba)b|(aa|b(a|b))a)b|(b(aa)|a(aa))a))))b|(b(a(a(b((bb)b|(ba|aa)a)|a((bb|a(a|b))a|(ab)b))|b(((aa)b|(bb|a(a|b))a)b|((ba|(a|b)b)a|(bb|ba)b)a))|b(a(b(a(aa)|b(bb|a(a|b)))|a((bb|a(a|b))(a|b)))|b(b(b(bb|a(a|b))|a(ab|bb))|a((bb|ba)(a|b)))))|a(b((a((bb)b|(ba|aa)a)|b(b(bb|a(a|b))|a(ab)))a|(a(b(bb)|a(aa|b(a|b)))|b((ab)a))b)|a((b((bb|a(a|b))b|(ba|(a|b)b)a)|a((ab)a|(aa|ab)b))a|(a(b(ba|aa)|a(aa))|b((ab)b|(ab)a))b)))a)((b(((b(b(bb|ba)|a(ab|bb))|a((aa|ab)(a|b)))b|((b(a(a|b)|ba)|a(bb))a|(a(bb|a(a|b))|b((a|b)(a|b)))b)a)a|(a((b(bb|a(a|b))|a(ab))a|(a(bb)|b(bb))b)|b(((ba|aa)a|(bb|a(a|b))b)a|((ab)b|(a(a|b)|ba)a)b))b)|a(b((b(b(aa)|a(aa))|a(a(ba|ab)|b(aa)))a|(a(b(ab)|a(ab|bb))|b(b(bb)|a((a|b)(a|b))))b)|a((b(b(bb|aa)|a(ba))|a((ab)a|(bb|aa)b))b|(a(a(aa)|b(bb|a(a|b)))|b(a(bb|aa)|b(aa|ab)))a)))b|((b((((aa|b(a|b))b|(ab)a)b|((aa|b(a|b))b|(ab|bb)a)a)b|((b(aa)|a(aa))b|(b(bb|ba)|a(ba))a)a)|a(a(((ab)b|(ab|bb)a)b|((ab)b|(a(a|b)|ba)a)a)|b(b((bb|a(a|b))a|(ab)b)|a((a(a|b)|ba)b|(ab|bb)a))))a|(b((a((aa|b(a|b))b|(ab|bb)a)|b(a(ba)|b(aa)))b|(((bb|aa)a|(a(a|b)|ba)b)a|((ba|aa)a|(aa)b)b)a)|a(((b(ba|(a|b)b)|a(bb|ba))a|(b(ab|bb)|a(bb|aa))b)a|(b((bb|ba)b|((a|b)(a|b))a)|a((bb|a(a|b))b|((a|b)(a|b))a))b))b)a))$
    


```python
rx = re.compile(r0)
#matches = [bool(rx.match(x)) for x in test_dat2]
matches = [bool(rx.match(x)) for x in dat]
#matches
```


```python
Markdown("The number of messages that completely match rule 0 is "
         "**{}**".format(sum(matches)))
```




The number of messages that completely match rule 0 is **269**



## Part Two

As you look over the list of messages, you realize your matching rules aren't quite right. To fix them, completely replace rules 8: 42 and 11: 42 31 with the following:
```
8: 42 | 42 8
11: 42 31 | 42 11 31
```
This small change has a big impact: now, the rules do contain loops, and the list of messages they could hypothetically match is infinite. You'll need to determine how these changes affect which messages are valid.

Fortunately, many of the rules are unaffected by this change; it might help to start by looking at which rules always match the same set of values and how those rules (especially rules 42 and 31) are used by the new versions of rules 8 and 11.

(Remember, you only need to handle the rules you have; building a solution that could handle any hypothetical combination of rules would be significantly more difficult.)

For example:
```
42: 9 14 | 10 1
9: 14 27 | 1 26
10: 23 14 | 28 1
1: "a"
11: 42 31
5: 1 14 | 15 1
19: 14 1 | 14 14
12: 24 14 | 19 1
16: 15 1 | 14 14
31: 14 17 | 1 13
6: 14 14 | 1 14
2: 1 24 | 14 4
0: 8 11
13: 14 3 | 1 12
15: 1 | 14
17: 14 2 | 1 7
23: 25 1 | 22 14
28: 16 1
4: 1 1
20: 14 14 | 1 15
3: 5 14 | 16 1
27: 1 6 | 14 18
14: "b"
21: 14 1 | 1 14
25: 1 1 | 1 14
22: 14 14
8: 42
26: 14 22 | 1 20
18: 15 15
7: 14 5 | 1 21
24: 14 1

abbbbbabbbaaaababbaabbbbabababbbabbbbbbabaaaa
bbabbbbaabaabba
babbbbaabbbbbabbbbbbaabaaabaaa
aaabbbbbbaaaabaababaabababbabaaabbababababaaa
bbbbbbbaaaabbbbaaabbabaaa
bbbababbbbaaaaaaaabbababaaababaabab
ababaaaaaabaaab
ababaaaaabbbaba
baabbaaaabbaaaababbaababb
abbbbabbbbaaaababbbbbbaaaababb
aaaaabbaabaaaaababaa
aaaabbaaaabbaaa
aaaabbaabbaaaaaaabbbabbbaaabbaabaaa
babaaabbbaaabaababbaabababaaab
aabbbbbaabbbaaaaaabbbbbababaaaaabbaaabba
```
Without updating rules 8 and 11, these rules only match three messages: bbabbbbaabaabba, ababaaaaaabaaab, and ababaaaaabbbaba.

However, after updating rules 8 and 11, a total of 12 messages match:

    bbabbbbaabaabba
    babbbbaabbbbbabbbbbbaabaaabaaa
    aaabbbbbbaaaabaababaabababbabaaabbababababaaa
    bbbbbbbaaaabbbbaaabbabaaa
    bbbababbbbaaaaaaaabbababaaababaabab
    ababaaaaaabaaab
    ababaaaaabbbaba
    baabbaaaabbaaaababbaababb
    abbbbabbbbaaaababbbbbbaaaababb
    aaaaabbaabaaaaababaa
    aaaabbaabbaaaaaaabbbabbbaaabbaabaaa
    aabbbbbaabbbaaaaaabbbbbababaaaaabbaaabba

After updating rules 8 and 11, how many messages completely match rule 0?


```python
rules_raw2 = rules_raw.copy()

#rules_raw2[8] = '42 | 42 8'
# Expanding out how this would look: 42 | 42 42 | 42 42 42 | 42 42 42 42 ...
# So we can eliminate the infinite loop by just using the (+) regex expression
# for "match one or more". Our new rule then becomes:
rules_raw2[8] = '42+'

#rules_raw2[11] = '42 31 | 42 11 31'
# Expanding: 42 31 | 42 42 31 31 | 42 42 42 31 31 31 ...
# I don't know how to convert this to a regex... but if I just
# manually right out the first half-dozen cycles, maybe that will
# be enough:
rules_raw2[11] = ('42 31 | 42 42 31 31 | 42 42 42 31 31 31 | 42 42 42 42 31 31 31 31'
                  ' | 42 42 42 42 42 31 31 31 31 31 | 42 42 42 42 42 42 31 31 31 31 31 31')
```


```python
# Rule to match space not between digits
rnds = re.compile('(?<=\D)\s(?=\D)')

# Function for printing out choice rules well formatted
def print_rules(rule_list):
    irules = [0, 8, 11, 42, 31]
    print('=-'*20)
    for i in irules:
        r = rule_list[i]
        r = rnds.sub('', r)
        print("{}: {}".format(i, r))
```


```python
#rules_deref = rules_raw2.copy()
rules_deref = rules_raw2.copy()
skipset = set()
last_skipset = set()
#while len(skipset) < len(rules_deref):
keep_going = True
#print_rules(rules_deref)
while(keep_going):
    rules_deref, skipset = rep_rules(rules_deref, skipset)
    #print_rules(rules_deref)
    r0 = rules_deref[0]
    keep_going = bool(rdig.search(r0))
    if last_skipset == skipset:
        print("Did not get any new matches")
        keep_going = False
    last_skipset = skipset.copy()
#print(rules_deref[0])
r0 = '^' + rules_deref[0].replace(' ', '') + '$'
#print(r0)
```


```python
rx = re.compile(r0)
#matches = [bool(rx.match(x)) for x in test_dat2]
matches = [bool(rx.match(x)) for x in dat]
#matches
```


```python
Markdown("The number of messages that completely match rule 0 is "
         "**{}**".format(sum(matches)))
```




The number of messages that completely match rule 0 is **403**



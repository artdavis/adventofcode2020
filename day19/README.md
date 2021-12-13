```python
import re
from IPython.display import Markdown
from IPython.core.debugger import set_trace as breakpt
```

# Day 19: Monster Messages

Reference: https://adventofcode.com/2020/day/19

## Part 1

**How many messages completely match rule 0?**


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
#Markdown("The number of messages that completely match rule 0 is "
#         "**{}**".format(sum(matches)))
```

## Part Two

**After updating rules 8 and 11, how many messages completely match rule 0?**


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
#Markdown("The number of messages that completely match rule 0 is "
#         "**{}**".format(sum(matches)))
```


```python

```

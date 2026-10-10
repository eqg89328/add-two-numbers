# add two numbers: how the LeetCode 2 linked-list solution works, plus the edge cases and carry logic that make it stick

Search "add two numbers" and you get two completely different crowds. One wants the arithmetic explanation for a first grader. The other is staring down LeetCode problem 2, where two linked lists hold digits in reverse order and your job is to return their sum as a third list.

The second crowd is much larger, and it's who this article is for. If you're here for 342 + 465 = 807 with no pointers involved, you already know the answer. If you're here because you can read the solution and still can't reproduce it twenty minutes later, that gap is the actual problem worth fixing.

## What the problem is really asking

You get two non-empty linked lists. Each node holds a single digit from 0 to 9. The lists store digits in reverse order, so the head is the ones place, and the lengths run from 1 to 100 nodes. No leading zeros except for the number 0 itself.


Input:  l1 = [2,4,3], l2 = [5,6,4]
Output: [7,0,8]


Read the lists backwards and you get 342 and 465. Add them and you get 807, which comes back as 7 → 0 → 8. That reverse storage is not a curveball, it's a favor. Addition starts at the ones place and moves left, and the list already points you that way, so you never need to reverse anything or know the total length in advance.

The constraints matter more than they look. With up to 100 digits per list, you cannot convert to an integer and add. In Python that would technically work, but in Java or C++ a 100-digit number blows past `long` fast, and an interviewer who sees `Integer.parseInt` will ask you what happens at digit 19. The intended solution never converts anything.

## 342 + 465, one digit at a time

Before writing a line of code, walk the example by hand. Three things travel through the loop: a pointer into each list, and a carry.

| Step | l1 digit | l2 digit | Carry in | Sum | Digit written | Carry out |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 2 | 5 | 0 | 7 | 7 | 0 |
| 2 | 4 | 6 | 0 | 10 | 0 | 1 |
| 3 | 3 | 4 | 1 | 8 | 8 | 0 |

The result list is 7 → 0 → 8. The only interesting moment is step 2, where 4 + 6 = 10. You write down 0 and carry 1 into the next column, exactly like pencil-and-paper addition in school. Every carry in the problem is either 0 or 1, because the largest possible sum per column is 9 + 9 + 1 = 19.

## The solution: a dummy head, a carry, one loop

Two implementation details do most of the work.

The first is a dummy head node. You create a placeholder, keep a moving pointer to it, and build the result behind it. When the loop finishes you return `dummy.next`. Without this, you need a special branch for the first node, and that branch is where most wrong answers come from.

The second is the loop condition: keep going while either list still has a node *or* a carry remains. That third clause is what handles 9999999 + 9999, where the answer has one more digit than the longer input.

python
class Solution:
    def addTwoNumbers(self, l1, l2):
        dummy = ListNode(0)
        cur = dummy
        carry = 0

        while l1 or l2 or carry:
            total = (l1.val if l1 else 0) + (l2.val if l2 else 0) + carry
            carry, digit = divmod(total, 10)
            cur.next = ListNode(digit)
            cur = cur.next
            l1 = l1.next if l1 else None
            l2 = l2.next if l2 else None

        return dummy.next


The same structure in Java, since that's the language most people get asked this in:

java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0);
        ListNode cur = dummy;
        int carry = 0;

        while (l1 != null || l2 != null || carry != 0) {
            int sum = carry;
            if (l1 != null) { sum += l1.val; l1 = l1.next; }
            if (l2 != null) { sum += l2.val; l2 = l2.next; }
            carry = sum / 10;
            cur.next = new ListNode(sum % 10);
            cur = cur.next;
        }
        return dummy.next;
    }
}


AlgoMonster hosts an editorial for this exact problem with Python and Java implementations alongside the complexity notes, and it's part of the linked-list material rather than standing alone.

## The edge cases people actually fail

**The carry that outlives both lists.** Input `[9,9,9,9,9,9,9]` and `[9,9,9,9]` is 9999999 + 9999 = 10009998, which returns `[8,9,9,9,0,0,0,1]`. If your loop is `while (l1 != null && l2 != null)`, you drop the final 1 and fail the test. Using `||` plus a carry check costs nothing and closes that hole.

**Unequal lengths.** When one list runs out, its digit is 0, not "stop the loop." Treating a null node as zero is what lets `while (l1 != null || l2 != null)` work at all.

**The zero case.** `[0] + [0]` returns `[0]`. The dummy head setup handles it without any special handling, which is a good sign you've structured things correctly.

**Overflow by conversion.** Turning both lists into ints and adding is the naive solution an interviewer expects you to propose and then reject. Constraints allow 100 digits, so it fails by design.

One more that's easy to miss: you modify `l1` and `l2` as you traverse, so if the caller needs the original lists, you'd want local copies. Most judges don't care, but it's the kind of follow-up question that comes up.

## Complexity

Time is O(max(m, n)), where m and n are the list lengths. You touch each node once and do constant work per node.

Auxiliary space is O(1). The result list itself is O(max(m, n)) and isn't avoidable, but the algorithm adds no extra structures beyond a couple of pointers and an int. If you find yourself allocating a stack or an array, you've added work the problem doesn't require.

## Which pattern this belongs to, and why it matters

Add Two Numbers is not a two-pointer problem in the sliding-window sense, despite the name. It's linked-list traversal with a running state variable, in this case the carry. That's a small but real distinction, because the pattern you should walk away with is: *traverse two structures in lockstep, keep an accumulator, handle mismatched lengths with a default value.* Once you see it that way, a family of problems opens up.

- **Add Two Numbers II** (LeetCode 445) gives you the digits in normal order, so the same loop needs a reversal or an explicit stack first.
- **Add Strings** and **Add Binary** are the same carry loop over character arrays instead of nodes.
- **Multiply Strings** is the carry loop nested inside another loop.
- **Plus One** on a linked list is the carry loop with a single input.

Here is the thing worth sitting with. If you memorize the code above, you can solve exactly one problem. If you internalize "lockstep traversal plus carry," you can solve the five related ones and recognize the variation before the interviewer finishes the question. Interviewers rarely test whether you know one problem. They test whether you can spot that an unfamiliar problem is a familiar shape.

That's the argument behind pattern-based prep generally, and it's the specific bet AlgoMonster makes: teach roughly 48 patterns with templates and decision flowcharts, then let the problems be practice material instead of the syllabus.

## Where AlgoMonster fits if you're working through problems like this

AlgoMonster is a text-and-diagram interview prep platform built by ex-Google engineers and competitive programmers. No video. You read, you look at animations, you write code in the browser and get feedback.

What's on the platform, per its current subscribe page:

- 500+ lessons across 6 courses, with 1,000+ illustrations and 3,000+ editorials
- Reusable code templates per pattern, in multiple languages
- Decision flowcharts for figuring out which pattern a new problem needs
- Speedrun mode for timed review passes
- AI assistant woven through the content: selected-text explanations, code debugging on submission, and tracking of the mistakes you repeat
- Company-tagged questions refreshed monthly
- Two bonus courses included: a CS foundation course and a build-your-own-LLM course
- A member discount on live bootcamps

On pricing, one thing to know upfront: the listed price moves constantly. AlgoMonster runs near-permanent sales, and the yearly and lifetime figures on the subscribe page change with the promo running.

## AlgoMonster plans and current pricing

| Plan | What you get | Price | Billing | Get it |
| --- | --- | --- | --- | --- |
| Free | Limited access to a sample of lessons and problems | $0 | — | [ Start with the free plan](https://algo.monster?fpr=li69) |
| Pro – Yearly | Everything unlocked: all patterns, lessons and problems, unlimited AI assistant, unlimited Speedrun, company questions, bonus courses, monthly content updates | $99/year (shown as $8.25/mo billed annually) — has appeared as low as $69.99/year during deeper promotions | Annual, cancels or upgrades anytime | [ Get Pro Yearly](https://algo.monster?fpr=li69) |
| Pro – Lifetime | Everything in the yearly plan plus lifetime updates, one payment, no renewals | $189 one-time — has appeared as low as $89 during deeper promotions | One-time | [ Get Pro Lifetime](https://algo.monster?fpr=li69) |

All three are on the same subscribe page, and the promo price is applied at checkout. Note that specialized courses built with external partners, such as Quant Interview Prep, are sold separately and aren't included in either Pro tier.

If you're comparing against what you already pay: LeetCode Premium runs about $159/year, and AlgoMonster's yearly plan is currently listed below that. The trade isn't price, it's volume. AlgoMonster has a few hundred problems mapped to patterns. LeetCode has several thousand. Most people preparing seriously end up using both, with AlgoMonster first for the framework and LeetCode after for speed.

## Which plan, and whether you're the right buyer

**Yearly makes sense if** you have an interview loop in the next two to three months, you can already write working code in one language, and you want the pattern framework without a long-term commitment. The curriculum is designed to be worked through in roughly four to eight weeks at an hour or two a day, which fits inside that window.

**Lifetime makes sense if** you expect to interview again. If you're going to change jobs every couple of years, and each search sends you back to brushing up on pattern recognition, paying once removes the renewal question entirely. The break-even against annual renewals is roughly two years, so anyone confident about a second or third search later comes out ahead.

**Skip it if** you're two weeks from an onsite (there's not enough runway for the material to stick), you're still learning programming fundamentals rather than problem selection, or you're targeting senior roles where the interview is mostly system design. AlgoMonster's system design content is introductory, and its behavioral content is thin. It also runs on reading and diagrams, so if video walkthroughs are how your brain works, you'll get less out of it than someone who prefers text.

The honest framing: AlgoMonster is efficient at making you faster at identifying which approach a problem needs. It does not simulate interview pressure, and it doesn't put a senior engineer on the other side of your screen asking what changes if the input is streamed rather than in memory. Those follow-up questions are where prepared candidates still lose offers, and no practice platform closes that gap for you.

## Quick answers

**Do I need to know linked lists before starting this problem?** Yes. You need to be comfortable with node references, `.next` traversal, and constructing a list. Add Two Numbers is a linked list problem that happens to involve arithmetic, not an arithmetic problem that happens to involve lists.

**Why reverse order instead of normal order?** Because addition produces carries that propagate leftward, toward more significant digits. Reverse storage means the carry always moves forward through the list. Version II of the problem flips the order specifically to make it harder.

**What's the carry value at most?** 1. Digits cap at 9, so 9 + 9 + 1 = 19. If you're storing a carry larger than 1, your update logic is wrong.

**Can I do it recursively?** Yes, and it reads elegantly, but it uses O(max(m, n)) stack space instead of O(1). For a 100-node list that's fine in practice, and it's a reasonable thing to mention after giving the iterative version.

**Is the free tier enough to judge the platform?** It's enough to see how the content is written and whether text-plus-diagrams suits you. It isn't enough to prepare from. The paid plans are where the pattern coverage, AI assistant, and Speedrun mode live.

Work the naive approach first, state why it fails on 100-digit inputs, then build the dummy head and carry loop. That's the version an interviewer wants to hear, and it's the version you should be able to write from scratch — no reference, no tab open — before you move on to the next problem.

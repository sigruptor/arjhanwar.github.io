## LinkedList

### Reversing
There are multiple ways to reverse a LinkedList.

#### Iterative
```
    private ListNode iterativeReverseList(ListNode head) {
        ListNode newHead = null;
        ListNode prev = null, next = null;
        while (head!=null) {
            next = head.next;
            head.next = prev;
            prev = head;
            head = next;
        }
        return prev;
    }
```

#### Recursive

```
    private ListNode recursiveReverseList(ListNode head) {
        if (head == null || head.next == null) {
            return null;
        }
        ListNode reverseHead = recursiveReverseList(head.next);
        head.next.next = head;
        head.next = null;
        return reverseHead;
    }
```

With an additional parameter
```
    private ListNode reverseList(ListNode head, ListNode prev) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode next = head.next;
        head.next = prev;

        return reverseList(next, head);
    }
```

With a Global Class level variable
```
    public ListNode reverseList(ListNode head) {
        newHead = null;
        reverse(head);
        return newHead;
    }
    
    public ListNode reverse(ListNode head) {
        if (head == null || head.next == null) {
            newHead = head;
            return head;
        }
        ListNode rev = reverse(head.next);
        rev.next = head;
        head.next = null;
        return head;
    }

```

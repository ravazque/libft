
# ft_strlcat - How It Works

## Prototype

```c
size_t ft_strlcat(char *dst, const char *src, size_t dstsize)
```

## What the function does

Concatenates src to the end of dst without exceeding dstsize, ensuring dst always ends with '\0'.

The return value is ALWAYS `strlen(initial_dst) + strlen(src)`, regardless of how much was actually copied.

## The initial check

```c
if (lngdst >= dstsize)
    return (dstsize + lngsrc);
```

### Why this check exists

This check detects two problematic situations:

**Situation 1: Improperly sized buffer**
```c
char dst[10] = "Hello world complete";  // 20 characters in a 10-byte buffer
ft_strlcat(dst, "extra", 10);
```
Here `lngdst = 20` but `dstsize = 10`. This is impossible: the string cannot be longer than its own buffer. The function detects something is wrong.

**Situation 2: Completely full buffer**
```c
char dst[10] = "123456789";  // 9 characters + '\0' = 10 bytes (full buffer)
ft_strlcat(dst, "extra", 10);
```
Here `lngdst = 9` and `dstsize = 10`. There's no room to add anything because the only free byte is occupied by the final '\0'.

### Why it returns dstsize + lngsrc

When `lngdst >= dstsize`, the function cannot know the real length of dst (it may be corrupted or mismeasured). Therefore it assumes dst has length `dstsize` and returns as if it were concatenating from there:

```
Return = dstsize + lngsrc
```

This allows the caller to detect that something failed by comparing the return with dstsize.

## How it concatenates

```c
while (src[i] != '\0' && (lngdst + i) < (dstsize - 1))
{
    dst[lngdst + i] = src[i];
    i++;
}
dst[lngdst + i] = '\0';
```

### The while condition explained

The while has two conditions that must be met simultaneously:

**Condition 1: `src[i] != '\0'`**

We haven't reached the end of src. Simple.

**Condition 2: `(lngdst + i) < (dstsize - 1)`**

Here's the key to safe operation. Let's break it down:

- `lngdst` is the position where writing starts (current end of dst)
- `i` is how many characters from src have been copied so far
- `lngdst + i` is the position where the next character will be written
- `dstsize - 1` is the last position where a normal character can be written

### Why dstsize - 1

The buffer has `dstsize` positions (indices 0 to dstsize-1). The last position MUST be '\0'. Therefore:

```
Positions available for normal characters: 0 to (dstsize - 2)
Position reserved for '\0': (dstsize - 1)
```

Example with dstsize = 10:

```
Indices:    [0][1][2][3][4][5][6][7][8][9]
Content:    [ ][ ][ ][ ][ ][ ][ ][ ][ ][\0]
                                        ^
                                        Always '\0'
```

You can only write normal characters in positions 0 to 8. Position 9 must be '\0'.

That's why the condition is `< (dstsize - 1)`:
- If I write at position 8, I can continue (8 < 9)
- If I try to write at position 9, it stops (9 is NOT < 9)

### Step-by-step example

```c
char dst[10] = "Hell";   // lngdst = 4, dstsize = 10
char src[] = "o world!"; // lngsrc = 8
```

Initial state:
```
dst: [H][e][l][l][\0][ ][ ][ ][ ][ ]
      0  1  2  3  4   5  6  7  8  9
```

Copy process:

```
i=0: lngdst + i = 4 + 0 = 4
     4 < 9 (dstsize-1) → TRUE
     Copy src[0]='o' to dst[4]
     dst: [H][e][l][l][o][\0][ ][ ][ ][ ]

i=1: lngdst + i = 4 + 1 = 5
     5 < 9 → TRUE
     Copy src[1]=' ' to dst[5]
     dst: [H][e][l][l][o][ ][\0][ ][ ][ ]

i=2: 4 + 2 = 6 < 9 → TRUE, copy 'w'
i=3: 4 + 3 = 7 < 9 → TRUE, copy 'o'
i=4: 4 + 4 = 8 < 9 → TRUE, copy 'r'

i=5: lngdst + i = 4 + 5 = 9
     9 < 9 → FALSE
     STOP: doesn't copy 'l', 'd', or '!'
```

Final state before adding '\0':
```
dst: [H][e][l][l][o][ ][w][o][r][ ]
      0  1  2  3  4  5  6  7  8  9
                                i=5 (didn't copy)
```

Then adds '\0':
```c
dst[lngdst + i] = '\0';  // dst[4 + 5] = dst[9] = '\0'
```

Final result:
```
dst: [H][e][l][l][o][ ][w][o][r][\0]
      0  1  2  3  4  5  6  7  8  9
```

## Why the return is different from what was copied

```c
return (lngdst + lngsrc);  // Returns 4 + 8 = 12
```

In the previous example:
- Only copied 5 characters from src (o, space, w, o, r)
- But returns 12 (which would be the length if everything was copied)

This allows truncation detection:

```c
size_t result = ft_strlcat(dst, src, 10);

if (result >= 10) {
    // Truncated: result=12, dstsize=10
    // You'd need a buffer of at least 13 bytes (12 + '\0')
}
```

## Summary of operation

1. Calculate lengths of dst and src
2. If dst is already full or buffer is invalid, return without copying
3. Copy characters from src to dst one by one
4. Stop when:
   - src ends, OR
   - Next position would be dstsize-1 (reserved for '\0')
5. Place '\0' where it stopped
6. Return the total length it would have had (initial_dst + complete src)

The design guarantees that:
- Never exceeds dstsize
- dst always ends with '\0'
- The caller can know if truncation occurred

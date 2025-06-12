# Libft

![Libft](https://raw.githubusercontent.com/ayogun/42-project-badges/main/badges/libfte.png)

*My very first own library*

## 📖 About

**Libft** is the first project at 42 Madrid. It involves creating a custom C library by recoding standard C library functions and implementing additional utility functions. This library serves as a foundation for future projects throughout the 42 curriculum, as the use of external libraries is restricted.

The project is divided into three main parts:
- **Part I**: Re-implementation of standard libc functions
- **Part II**: Additional utility functions not present in libc
- **Bonus**: Implementation of functions for linked list manipulation

## 🎯 Objectives

- Deep understanding of C programming fundamentals
- Memory management and pointer manipulation
- Understanding of standard library function implementations
- Creating a static library using Makefile
- Introduction to data structures (linked lists)

## 🛠️ Functions Implemented

<details>
<summary><strong>Part I - Libc Functions</strong></summary>

### Character Classification & Conversion
- `ft_isalpha` - Check if character is alphabetic
- `ft_isdigit` - Check if character is a digit
- `ft_isalnum` - Check if character is alphanumeric
- `ft_isascii` - Check if character is ASCII
- `ft_isprint` - Check if character is printable
- `ft_toupper` - Convert to uppercase
- `ft_tolower` - Convert to lowercase

### String Manipulation
- `ft_strlen` - Calculate length of string
- `ft_strchr` - Locate character in string
- `ft_strrchr` - Locate character in string (reverse)
- `ft_strncmp` - Compare strings up to n characters
- `ft_strlcpy` - Copy string with size limit
- `ft_strlcat` - Concatenate strings with size limit
- `ft_strnstr` - Locate substring in string

### Memory Functions
- `ft_memset` - Fill memory with constant byte
- `ft_bzero` - Zero a byte string
- `ft_memcpy` - Copy memory area
- `ft_memmove` - Copy memory area (handles overlap)
- `ft_memchr` - Scan memory for character
- `ft_memcmp` - Compare memory areas

### Conversion
- `ft_atoi` - Convert string to integer
- `ft_calloc` - Allocate and zero memory
- `ft_strdup` - Duplicate string

</details>

<details>
<summary><strong>Part II - Additional Functions</strong></summary>

- `ft_substr` - Extract substring from string
- `ft_strjoin` - Concatenate two strings
- `ft_strtrim` - Trim characters from beginning and end
- `ft_split` - Split string using delimiter
- `ft_itoa` - Convert integer to string
- `ft_strmapi` - Apply function to each character with index
- `ft_striteri` - Apply function to each character (in place)
- `ft_putchar_fd` - Output character to file descriptor
- `ft_putstr_fd` - Output string to file descriptor
- `ft_putendl_fd` - Output string with newline to file descriptor
- `ft_putnbr_fd` - Output number to file descriptor

</details>

<details>
<summary><strong>Bonus - Linked List Functions</strong></summary>

- `ft_lstnew` - Create new list element
- `ft_lstadd_front` - Add element at beginning of list
- `ft_lstsize` - Count elements in list
- `ft_lstlast` - Return last element of list
- `ft_lstadd_back` - Add element at end of list
- `ft_lstdelone` - Delete single element
- `ft_lstclear` - Delete and free list
- `ft_lstiter` - Apply function to each element
- `ft_lstmap` - Apply function and create new list

</details>

## 🚀 Installation & Usage

### Compilation

```bash
# Clone the repository
git clone https://github.com/ravazque/libft.git
cd libft

# Compile the library
make

# Compile with bonus functions
make bonus

# Clean object files
make clean

# Clean everything including library
make fclean

# Recompile everything
make re
```

### Usage Example

```c
#include "libft.h"
#include <stdio.h>

int main(void)
{
    char *str = "Hello, 42!";
    char *substr;
    char **split_result;
    int i;

    // String manipulation
    printf("Original: %s\n", str);
    printf("Length: %zu\n", ft_strlen(str));
    
    // Extract substring
    substr = ft_substr(str, 7, 2);
    printf("Substring: %s\n", substr);
    free(substr);
    
    // Split string
    split_result = ft_split("apple,banana,orange", ',');
    i = 0;
    while (split_result[i])
    {
        printf("Split[%d]: %s\n", i, split_result[i]);
        free(split_result[i]);
        i++;
    }
    free(split_result);
    
    return (0);
}
```

### Compilation with your project

```bash
# Compile your project with libft
gcc -Wall -Wextra -Werror your_file.c -L. -lft -o your_program
```

## 📁 Project Structure

```
libft/
├── Makefile              # Compilation rules
├── libft.h              # Header file with function prototypes
├── ft_*.c               # Source files for each function
└── README.md            # Project documentation
```

## ⚡ Key Learning Points

- **Memory Management**: Understanding malloc, free, and memory leaks
- **Pointer Arithmetic**: Mastering pointer manipulation and dereferencing
- **String Handling**: Learning how strings work at a low level in C
- **Linked Lists**: Introduction to dynamic data structures
- **Makefile**: Understanding compilation process and library creation
- **Code Organization**: Structuring code for reusability and maintainability

## 🧪 Testing

The library has been thoroughly tested with:
- Custom test cases for edge cases and normal operations
- Memory leak testing with valgrind
- Comparison with original libc functions
- Stress testing with large inputs

## 💡 Implementation Notes

- All functions handle NULL pointers appropriately
- Memory allocation functions check for allocation failures
- String functions respect buffer boundaries
- Linked list functions maintain list integrity
- Code follows 42 coding standards (Norm)

## 🔧 Technical Specifications

- **Language**: C (C99 standard)
- **Compiler**: gcc with flags `-Wall -Wextra -Werror`
- **Memory Management**: Manual memory management with malloc/free
- **Library Type**: Static library (.a file)
- **Dependencies**: None (standard C library functions only for basic operations)

This library serves as the foundation for all subsequent 42 projects and demonstrates proficiency in low-level C programming concepts.

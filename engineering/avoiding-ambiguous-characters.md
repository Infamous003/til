# Avoid Ambiguous Characters in Generated Credentials

Problem:
Passwords & OTPs can contain confusing characters like:
- 0 and O
- I and l
- 1 and l

Solution:
Exclude ambiguous characters from the allowed alphabets

Example:
```go
const alphabet = "ABCDEFGHJKMNPQRSTUVWXYZabcdefghijkmnpqrstuvwxyz23456789"
```

Lesson:
This about the user experience, not just the functionality
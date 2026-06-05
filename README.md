# Daily Multi‑Language Code Streak Automator

One GitHub Action script that keeps your contribution streak alive by committing a random code snippet every day – different language per weekday.

## What it does

- Runs daily at 01:00 UTC on GitHub’s servers (no local machine needed).
- Checks the current weekday → maps it to a language (Mon=Python, Tue=JavaScript, Wed=Go, Thu=Rust, Fri=Java, Sat=TypeScript, Sun=C++).
- Picks a random valid code snippet from a pool of 3 per language.
- Creates a new file: `daily_codes/YYYY-MM-DD-language.ext`
- Commits with a dynamic message (Monday gets `"This is my commit for Monday"`, others get the date).
- Pushes the commit → your contribution graph gets a green square.

## Setup (3 steps)

1. **Create a public repo** (streaks require public or default branch of private on Pro).
2. **Add the workflow** – copy the YAML from below into `.github/workflows/keep-streak.yml`.
3. **Enable write permissions** – repo → Settings → Actions → General → Workflow permissions → **Read and write permissions**.

That’s it. The next scheduled run will happen automatically. You can also trigger it manually from the Actions tab.

## Customise easily

- **Change language per day** – edit the `case $WEEKDAY_NUM in` block inside the YAML.
- **Add more snippets** – increase `RAND_IDX % 3` to `% 4` and add another `case` entry.
- **Change commit message** – edit the `COMMIT_MSG` variable.
- **Change schedule (cron)** – modify the `cron:` line (use [crontab.guru](https://crontab.guru)).
- **Use your own name/email** – replace the two `git config` lines with your GitHub username and verified email.

## Important notes

- Repository **must be public** for free accounts – private repos only count commits on the default branch for GitHub Pro users.
- One commit per day is natural; don’t trigger it more often (looks fake, may get throttled).
- This is a learning/backup tool – real contributions matter more.

## How to stop

Delete `.github/workflows/keep-streak.yml` or disable the workflow in the Actions tab.

---

## The script (copy the entire block below)

Save as `.github/workflows/keep-streak.yml`:

```yaml
# ====================================================================
# GitHub Actions Workflow: Automated Multi‑Language Daily Code Commits
# ====================================================================
# Purpose: Every day, write a random code snippet in a different
#          programming language and commit it to keep your streak.
#          Each weekday gets a dedicated language (configurable).
# Runs on: GitHub cloud (no local machine needed)
# ====================================================================

name: "Daily Code Streak"

on:
  schedule:
    # Run daily at 01:00 UTC – adjust as you like
    - cron: "0 1 * * *"
  # Allow manual trigger from GitHub UI
  workflow_dispatch:

permissions:
  contents: write  # Required to push commits

jobs:
  generate-and-commit:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      # ================================================================
      # Main script: weekday → language → random snippet → commit
      # ================================================================
      - name: Generate daily code snippet
        run: |
          # ----------------------------------------
          # 1) Determine current weekday (Monday=1 ... Sunday=7)
          # ----------------------------------------
          WEEKDAY_NUM=$(date +%u)
          # Get full weekday name (e.g., "Monday")
          WEEKDAY_NAME=$(date +%A)

          # ----------------------------------------
          # 2) Map weekday to a primary language
          #    (You can freely change these assignments)
          # ----------------------------------------
          case $WEEKDAY_NUM in
            1) LANG="python"    ;;
            2) LANG="javascript";;
            3) LANG="go"        ;;
            4) LANG="rust"      ;;
            5) LANG="java"      ;;
            6) LANG="typescript";;
            7) LANG="cpp"       ;;
          esac

          # ----------------------------------------
          # 3) Build a dynamic commit message
          # ----------------------------------------
          # For Monday: "This is my commit for Monday"
          # Other days: "This is my automated commit for 2026-06-05"
          if [ "$WEEKDAY_NAME" = "Monday" ]; then
            COMMIT_MSG="This is my commit for $WEEKDAY_NAME"
          else
            COMMIT_MSG="This is my automated commit for $(date +%Y-%m-%d)"
          fi

          # ----------------------------------------
          # 4) Prepare file path (create folder if needed)
          #    Format: daily_codes/YYYY-MM-DD-<language>.ext
          # ----------------------------------------
          DATE_TAG=$(date +%Y-%m-%d)
          mkdir -p daily_codes

          # Determine file extension based on language
          case $LANG in
            python)     EXT="py" ;;
            javascript) EXT="js" ;;
            go)         EXT="go" ;;
            rust)       EXT="rs" ;;
            java)       EXT="java" ;;
            typescript) EXT="ts" ;;
            cpp)        EXT="cpp" ;;
          esac
          FILENAME="daily_codes/${DATE_TAG}-${LANG}.${EXT}"

          # ----------------------------------------
          # 5) Generate a random code snippet for the chosen language
          #    Each language has a small pool of valid, harmless programs.
          #    The script picks one randomly using $RANDOM.
          # ----------------------------------------
          # Use a random number between 0 and 999
          RAND_IDX=$(( RANDOM % 3 ))   # Each language has 3 variants

          case $LANG in
            python)
              case $RAND_IDX in
                0) cat > $FILENAME <<'EOF'
# Python: Fibonacci sequence
def fib(n):
    a, b = 0, 1
    for _ in range(n):
        print(a, end=' ')
        a, b = b, a + b
fib(10)
EOF
                ;;
                1) cat > $FILENAME <<'EOF'
# Python: Random greeting
import random
greetings = ["Hello", "Bonjour", "Hola", "Ciao", "こんにちは"]
print(f"{random.choice(greetings)}, GitHub! Today is $(date +%A)")
EOF
                ;;
                2) cat > $FILENAME <<'EOF'
# Python: FizzBuzz
for i in range(1, 31):
    if i % 15 == 0: print("FizzBuzz")
    elif i % 3 == 0: print("Fizz")
    elif i % 5 == 0: print("Buzz")
    else: print(i)
EOF
                ;;
              esac
              ;;

            javascript)
              case $RAND_IDX in
                0) cat > $FILENAME <<'EOF'
// JavaScript: Simple closure
function counter() {
    let count = 0;
    return () => ++count;
}
const c = counter();
console.log(c(), c(), c());
EOF
                ;;
                1) cat > $FILENAME <<'EOF'
// JavaScript: Array utilities
const nums = [5, 12, 8, 130, 44];
const filtered = nums.filter(n => n > 10);
console.log(`Numbers > 10: ${filtered}`);
EOF
                ;;
                2) cat > $FILENAME <<'EOF'
// JavaScript: Random HEX color
const randomColor = () => '#' + Math.floor(Math.random()*16777215).toString(16);
console.log(`Today's color: ${randomColor()}`);
EOF
                ;;
              esac
              ;;

            go)
              case $RAND_IDX in
                0) cat > $FILENAME <<'EOF'
// Go: Hello with weekday
package main
import "fmt"
import "time"
func main() {
    fmt.Printf("Automated commit on %s\\n", time.Now().Weekday())
}
EOF
                ;;
                1) cat > $FILENAME <<'EOF'
// Go: Simple goroutine
package main
import "fmt"
func main() {
    ch := make(chan string)
    go func() { ch <- "Hello from goroutine" }()
    fmt.Println(<-ch)
}
EOF
                ;;
                2) cat > $FILENAME <<'EOF'
// Go: Sum of squares
package main
import "fmt"
func main() {
    sum := 0
    for i := 1; i <= 10; i++ {
        sum += i * i
    }
    fmt.Printf("Sum of squares 1..10 = %d\\n", sum)
}
EOF
                ;;
              esac
              ;;

            rust)
              case $RAND_IDX in
                0) cat > $FILENAME <<'EOF'
// Rust: Factorial
fn factorial(n: u64) -> u64 {
    match n {
        0 | 1 => 1,
        _ => n * factorial(n - 1),
    }
}
fn main() {
    println!("5! = {}", factorial(5));
}
EOF
                ;;
                1) cat > $FILENAME <<'EOF'
// Rust: Vec iteration
fn main() {
    let v = vec![10, 20, 30];
    for i in &v {
        println!("{}", i);
    }
}
EOF
                ;;
                2) cat > $FILENAME <<'EOF'
// Rust: Simple struct
#[derive(Debug)]
struct Streak { days: u32 }
fn main() {
    let s = Streak { days: 7 };
    println!("Current streak: {:?}", s);
}
EOF
                ;;
              esac
              ;;

            java)
              case $RAND_IDX in
                0) cat > $FILENAME <<'EOF'
// Java: Main class with date
public class DailyCommit {
    public static void main(String[] args) {
        System.out.println("Commit date: " + java.time.LocalDate.now());
    }
}
EOF
                ;;
                1) cat > $FILENAME <<'EOF'
// Java: Simple lambda
import java.util.Arrays;
public class DailyCommit {
    public static void main(String[] args) {
        Arrays.asList("mon", "tue", "wed").forEach(day -> System.out.println(day));
    }
}
EOF
                ;;
                2) cat > $FILENAME <<'EOF'
// Java: Random number game
import java.util.Random;
public class DailyCommit {
    public static void main(String[] args) {
        int r = new Random().nextInt(100);
        System.out.println("Random number: " + r);
    }
}
EOF
                ;;
              esac
              ;;

            typescript)
              case $RAND_IDX in
                0) cat > $FILENAME <<'EOF'
// TypeScript: Type alias and interface
type Day = string;
interface Commit { date: Day; active: boolean }
const today: Commit = { date: new Date().toDateString(), active: true };
console.log(today);
EOF
                ;;
                1) cat > $FILENAME <<'EOF'
// TypeScript: Generic identity function
function identity<T>(arg: T): T { return arg; }
console.log(identity<string>("Streak safe"));
EOF
                ;;
                2) cat > $FILENAME <<'EOF'
// TypeScript: Array reduce
const nums: number[] = [1,2,3,4];
const sum = nums.reduce((a,b) => a + b, 0);
console.log(`Sum = ${sum}`);
EOF
                ;;
              esac
              ;;

            cpp)
              case $RAND_IDX in
                0) cat > $FILENAME <<'EOF'
// C++: Hello world with date
#include <iostream>
#include <ctime>
int main() {
    time_t now = time(0);
    std::cout << "Commit on " << ctime(&now);
    return 0;
}
EOF
                ;;
                1) cat > $FILENAME <<'EOF'
// C++: Simple lambda
#include <iostream>
int main() {
    auto greet = []() { std::cout << "Hello from C++\n"; };
    greet();
    return 0;
}
EOF
                ;;
                2) cat > $FILENAME <<'EOF'
// C++: Fibonacci (iterative)
#include <iostream>
int main() {
    int a=0, b=1, c;
    for(int i=0; i<10; i++) {
        std::cout << a << " ";
        c = a+b; a=b; b=c;
    }
    return 0;
}
EOF
                ;;
              esac
              ;;
          esac

          # ----------------------------------------
          # 6) Verify file was created, then git add/commit/push
          # ----------------------------------------
          if [ -f "$FILENAME" ]; then
            echo "✅ Generated: $FILENAME"
            git add "$FILENAME"
            git commit -m "$COMMIT_MSG"
            git push
          else
            echo "❌ Failed to generate code snippet"
            exit 1
          fi

      # End of main script
```

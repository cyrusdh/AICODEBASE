# Setting Up the Full Deluge Coding Interface

## Overview
The Full Deluge Coding Interface is Zoho's official free online simulator that provides a complete Deluge development environment without requiring any account registration.

## Access URLs

### 1. Main Deluge Editor (Try Now Interface)
**URL:** https://deluge.zoho.com/tryout
- Complete Deluge IDE simulation
- Real-time code execution
- No account required
- Full scripting environment

### 2. Interactive Learning Interface  
**URL:** https://deluge.zoho.com/learndeluge
- Step-by-step tutorials
- Interactive exercises
- Progressive learning path
- Built-in examples

## Setup Instructions

### Step 1: Access the Interface
1. Open your browser
2. Navigate to: **https://deluge.zoho.com/tryout**
3. The interface loads immediately - no login required

### Step 2: Interface Overview
The online editor includes:
- **Code Editor**: Main scripting area with syntax highlighting
- **Function Library**: Drag-and-drop function toolbar on the left
- **Execute Button**: Run your code instantly
- **Output Panel**: View results and debug information
- **Key Bindings**: Keyboard shortcuts for efficient coding

### Step 3: Basic Usage

#### Writing Your First Script
```deluge
// Simple "Hello World" example
message = "Hello, Deluge World!";
info message;
```

#### Testing Functions
```deluge
// Working with variables
name = "Developer";
greeting = "Welcome to Deluge, " + name + "!";
info greeting;

// Date operations
today = today();
info "Today's date: " + today;

// List operations
myList = {1, 2, 3, 4, 5};
info "List contains: " + myList;
```

#### Advanced Examples
```deluge
// Working with Maps
userInfo = Map();
userInfo.put("name", "John Doe");
userInfo.put("email", "john@example.com");
userInfo.put("role", "Developer");

info "User: " + userInfo.get("name");
info "Email: " + userInfo.get("email");

// Loops and conditions
for each item in {1, 2, 3, 4, 5}
{
    if(item % 2 == 0)
    {
        info item + " is even";
    }
    else
    {
        info item + " is odd";
    }
}
```

### Step 4: Using the Function Library

#### Drag-and-Drop Functions
- Browse categories: Text, Numbers, Date-Time, Collections
- Drag functions directly into your code
- Auto-complete syntax assistance
- Parameter hints and examples

#### Popular Function Categories:
- **Text Functions**: `toString()`, `substring()`, `length()`
- **Date Functions**: `today()`, `now()`, `addDays()`
- **Collection Functions**: `size()`, `contains()`, `sort()`
- **Math Functions**: `round()`, `ceiling()`, `floor()`

### Step 5: Testing and Debugging

#### Using Info Statements
```deluge
// Debug variables
testVar = "Debug me";
info "Value: " + testVar;

// Test conditions
score = 85;
if(score >= 80)
{
    info "Excellent!";
}
```

#### Error Handling
```deluge
try 
{
    result = 10 / 0;  // This will cause an error
    info result;
}
catch (ex)
{
    info "Error occurred: " + ex;
}
```

## Key Features

### 1. Real-Time Execution
- Instant code testing
- Immediate output display
- Error highlighting
- Syntax validation

### 2. No Setup Required
- Browser-based
- No downloads
- No account creation
- Cross-platform compatibility

### 3. Complete Function Library
- All Deluge functions available
- Interactive documentation
- Live examples
- Parameter guidance

### 4. Learning Tools
- Progressive tutorials
- Interactive exercises
- Code examples
- Best practices

## Best Practices for Using the Interface

### 1. Start Simple
```deluge
// Begin with basic operations
firstName = "John";
lastName = "Doe";
fullName = firstName + " " + lastName;
info "Full name: " + fullName;
```

### 2. Test Functions Individually
```deluge
// Test one function at a time
testDate = today();
info "Today: " + testDate;

futureDate = addDays(testDate, 30);
info "Future date: " + futureDate;
```

### 3. Use Info for Debugging
```deluge
// Debug complex logic
data = {10, 20, 30, 40, 50};
total = 0;

for each number in data
{
    total = total + number;
    info "Current total: " + total;  // Track progress
}

info "Final total: " + total;
```

### 4. Save Your Work Locally
Since this is a browser-based tool:
- Copy important scripts to local files
- Use your workspace for version control
- Document your test cases

## Advanced Usage Tips

### 1. Complex Data Structures
```deluge
// Working with nested maps
company = Map();
company.put("name", "Tech Corp");
company.put("employees", 100);

departments = List();
departments.add("Engineering");
departments.add("Marketing");
departments.add("Sales");

company.put("departments", departments);

info "Company: " + company.get("name");
info "Departments: " + company.get("departments");
```

### 2. Function Testing
```deluge
// Create reusable logic patterns
calculateTax(amount, rate)
{
    return amount * (rate / 100);
}

// Test the function
price = 1000;
taxRate = 8.5;
tax = calculateTax(price, taxRate);

info "Price: $" + price;
info "Tax: $" + tax;
info "Total: $" + (price + tax);
```

## Integration with Your Workspace

### Save Scripts Locally
1. Create a `deluge-scripts` folder in your workspace
2. Save tested scripts as `.deluge` or `.txt` files
3. Use Git for version control
4. Document your test results

### Example Workspace Structure:
```
/workspace/
├── deluge-scripts/
│   ├── basic-functions.deluge
│   ├── data-manipulation.deluge
│   ├── api-testing.deluge
│   └── test-results.md
└── projects/
```

## Next Steps

1. **Start with the learning interface**: https://deluge.zoho.com/learndeluge
2. **Practice in the full editor**: https://deluge.zoho.com/tryout
3. **Save your progress locally** in your workspace
4. **Move to Zoho CRM Sandbox** when ready for real-world testing

## Troubleshooting

### Common Issues:
- **Script not running**: Check for syntax errors
- **Unexpected output**: Use `info` statements for debugging
- **Function errors**: Verify parameter types and counts
- **Browser issues**: Try refreshing or use Chrome/Firefox

### Getting Help:
- Use the built-in function documentation
- Check syntax examples in the learning interface
- Test small code snippets first
- Use the Zoho community forums for complex issues

You're now ready to start coding with the Full Deluge Coding Interface! Begin with simple scripts and gradually work up to more complex scenarios.
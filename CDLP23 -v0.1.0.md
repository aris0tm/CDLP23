
## **Button → Command Flow:**

```
Frontend Button Click → API Call → Backend Route → Shell Command Execution
```

**Example Flow:**
1. **User clicks "Compile Program 3a"**
2. **Frontend sends:** `POST /api/compile/3a`
3. **Backend executes:**
   - `yacc -d ex3a.y`
   - `lex ex3a.l` 
   - `gcc lex.yy.c y.tab.c -o ex3a -ll -ly`
4. **Returns status** to frontend with real compilation results

##  **Key Backend Features:**

### **Hard-coded Program Mapping:**
```javascript
const PROGRAMS = {
    '3a': {
        lexFile: 'ex3a.l',
        yaccFile: 'ex3a.y', 
        executable: 'ex3a'
    }
    // ... all your programs mapped
};
```

### **Real Command Execution:**
- Uses Node.js `child_process.exec()` 
- Executes actual `yacc`, `lex`, `gcc` commands
- Captures real stdout/stderr output
- Handles compilation errors properly

### **API Endpoints:**
- `POST /api/compile/:programId` - Compiles the program
- `POST /api/execute/:programId` - Runs with user input
- `POST /api/clean/:programId` - Cleans build files
- `GET /api/programs` - Lists all available programs

##  **Quick Setup:**

1. **Create the project structure:**
```bash
mkdir lex-yacc-executor
cd lex-yacc-executor
mkdir programs public
```

2. **Install dependencies:**
```bash
npm init -y
npm install express cors
```

3. **Put your .l and .y files in `./programs/`**

4. **Start server:**
```bash
node server.js
```

5. **Open browser:** `http://localhost:3000`

##  **What You Get:**

- **Real compilation** with actual error messages
- **Interactive execution** with user input
- **Cross-platform support** (Windows/Linux commands)
- **Real-time logging** of all compilation steps
- **Status indicators** showing program state
- **Clean error handling** and file cleanup

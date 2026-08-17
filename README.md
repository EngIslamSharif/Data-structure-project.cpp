# Data-structure-project.cpp


#include <iostream>
#include <fstream>
#include <cmath>
using namespace std;

struct Element {
    bool isOp;
    double num;
    char op;
};

struct Stack {
    Element* arr;
    int top;
    int cap;
};

Stack createStack(int capacity) {
    Stack s;
    s.arr = new Element[capacity];
    s.top = -1;
    s.cap = capacity;
    return s;
}

bool isEmpty(Stack s) {
    return s.top == -1;
}

bool isFull(Stack s) {
    return s.top == s.cap - 1;
}

void expand(Stack& s) {
    int newCap = s.cap * 2;
    Element* newArr = new Element[newCap];
    for (int i = 0; i <= s.top; i++) {
        newArr[i] = s.arr[i];
    }
    delete[] s.arr;
    s.arr = newArr;
    s.cap = newCap;
}

void push(Stack& s, Element e) {
    if (isFull(s)) {
        expand(s);
    }
    s.top++;
    s.arr[s.top] = e;
}

Element pop(Stack& s) {
    Element err;
    err.isOp = true;
    err.num = 0;
    err.op = '\0';
    if (isEmpty(s)) {
        return err;
    }
    return s.arr[s.top--];
}

Element peek(Stack s) {
    Element err;
    err.isOp = true;
    err.num = 0;
    err.op = '\0';
    if (isEmpty(s)) {
        return err;
    }
    return s.arr[s.top];
}

bool isOperator(char c) {
    return c == '+' || c == '-' || c == '*' || c == '/' || c == '^';
}

int getPrecedence(char op) {
    if (op == '^') return 3;
    if (op == '*' || op == '/') return 2;
    if (op == '+' || op == '-') return 1;
    return 0;
}

double calc(double a, double b, char op) {
    if (op == '+') return a + b;
    if (op == '-') return a - b;
    if (op == '*') return a * b;
    if (op == '/') return a / b;
    if (op == '^') return pow(a, b);
    return 0;
}

void printEl(Element e) {
    if (e.isOp) {
        cout << e.op;
    } else {
        cout << e.num;
    }
}

void writeEl(Element e, ofstream& f) {
    if (e.isOp) {
        f << e.op;
    } else {
        f << e.num;
    }
}

Element readElement() {
    char c;
    cin >> c;

    if (c == '(' || c == ')') {
        Element e;
        e.isOp = true;
        e.num = 0;
        e.op = c;
        return e;
    }

    if (isOperator(c)) {
        if (c == '-') {
            char next = cin.peek();
            if (next >= '0' && next <= '9') {
                cin.putback(c);
                double n;
                cin >> n;
                Element e;
                e.isOp = false;
                e.num = n;
                e.op = '\0';
                return e;
            }
        }
        Element e;
        e.isOp = true;
        e.num = 0;
        e.op = c;
        return e;
    }

    if ((c >= '0' && c <= '9') || c == '.') {
        cin.putback(c);
        double n;
        cin >> n;
        Element e;
        e.isOp = false;
        e.num = n;
        e.op = '\0';
        return e;
    }

    cout << "Error: Invalid input '" << c << "'!" << endl;
    Element e;
    e.isOp = true;
    e.num = 0;
    e.op = '?';
    return e;
}

void infixToPostfix(Stack& userStack, ofstream& out) {
    Stack opStack = createStack(userStack.top + 1);

    Element* postfix = new Element[userStack.top + 1];
    int pCount = 0;
    bool error = false;

    for (int i = 0; i <= userStack.top && !error; i++) {
        Element e = userStack.arr[i];

        if (!e.isOp) {
            postfix[pCount++] = e;
        }
        else if (e.op == '(') {
            push(opStack, e);
        }
        else if (e.op == ')') {
            while (!isEmpty(opStack) && peek(opStack).op != '(') {
                postfix[pCount++] = pop(opStack);
            }
            if (isEmpty(opStack)) {
                cout << "Error: Mismatched parentheses!" << endl;
                out << "Error: Mismatched parentheses!" << endl;
                error = true;
            } else {
                pop(opStack);
            }
        }
        else {
            while (!isEmpty(opStack) && peek(opStack).op != '(' &&
                ((e.op != '^' && getPrecedence(peek(opStack).op) >= getPrecedence(e.op)) ||
                    (e.op == '^' && getPrecedence(peek(opStack).op) > getPrecedence(e.op)))) {
                postfix[pCount++] = pop(opStack);
            }
            push(opStack, e);
        }
    }

    while (!isEmpty(opStack) && !error) {
        Element e = pop(opStack);
        if (e.op == '(') {
            cout << "Error: Mismatched parentheses!" << endl;
            out << "Error: Mismatched parentheses!" << endl;
            error = true;
        } else {
            postfix[pCount++] = e;
        }
    }

    if (error) {
        delete[] postfix;
        return;
    }

    Stack evalStack = createStack(pCount + 1);

    for (int i = 0; i < pCount && !error; i++) {
        if (!postfix[i].isOp) {
            push(evalStack, postfix[i]);
        } else {
            if (evalStack.top < 1) {
                cout << "Error: Invalid expression!" << endl;
                out << "Error: Invalid expression!" << endl;
                error = true;
                break;
            }
            double b = pop(evalStack).num;
            double a = pop(evalStack).num;

            if (postfix[i].op == '/' && b == 0) {
                cout << "Error: Division by zero!" << endl;
                out << "Error: Division by zero!" << endl;
                error = true;
                break;
            }

            double result = calc(a, b, postfix[i].op);
            Element r;
            r.isOp = false;
            r.num = result;
            r.op = '\0';
            push(evalStack, r);
        }
    }

    cout << "Postfix: ";
    out << "Postfix: ";
    for (int i = 0; i < pCount; i++) {
        printEl(postfix[i]);
        cout << " ";
        writeEl(postfix[i], out);
        out << " ";
    }
    cout << endl;
    out << endl;

    if (!error && evalStack.top == 0) {
        cout << "Result: " << evalStack.arr[0].num << endl;
        out << "Result: " << evalStack.arr[0].num << endl;
    } else if (!error) {
        cout << "Error: Invalid expression!" << endl;
        out << "Error: Invalid expression!" << endl;
    }

    delete[] postfix;
}

void evalPostfix(Stack& userStack, ofstream& out) {
    Stack evalStack = createStack(userStack.top + 1);
    bool error = false;

    for (int i = 0; i <= userStack.top && !error; i++) {
        Element e = userStack.arr[i];

        if (!e.isOp) {
            push(evalStack, e);
        } else {
            if (evalStack.top < 1) {
                cout << "Error: Invalid expression!" << endl;
                out << "Error: Invalid expression!" << endl;
                error = true;
                break;
            }
            double b = pop(evalStack).num;
            double a = pop(evalStack).num;

            if (e.op == '/' && b == 0) {
                cout << "Error: Division by zero!" << endl;
                out << "Error: Division by zero!" << endl;
                error = true;
                break;
            }

            double result = calc(a, b, e.op);
            Element r;
            r.isOp = false;
            r.num = result;
            r.op = '\0';
            push(evalStack, r);
        }
    }

    if (!error && evalStack.top == 0) {
        cout << "Result: " << evalStack.arr[0].num << endl;
        out << "Result: " << evalStack.arr[0].num << endl;
    } else if (!error) {
        cout << "Error: Invalid expression!" << endl;
        out << "Error: Invalid expression!" << endl;
    }
}

int main() {
    Stack s;
    s.arr = NULL;
    s.top = -1;
    s.cap = 0;
    bool created = false;

    ofstream out("output.txt");

    int choice;
    do {
        cout << "\n========================" << endl;
        cout << "         MENU           " << endl;
        cout << "========================" << endl;
        cout << "1. Create Stack" << endl;
        cout << "2. Push Element" << endl;
        cout << "3. Pop Element" << endl;
        cout << "4. Infix -> Postfix + Evaluate" << endl;
        cout << "5. Evaluate Postfix (Single Digit)" << endl;
        cout << "6. Evaluate Postfix (Multi Digit)" << endl;
        cout << "7. Exit" << endl;
        cout << "========================" << endl;
        cout << "Enter choice: ";
        cin >> choice;

        switch (choice) {

        case 1: {
            int size;
            cout << "Enter stack capacity: ";
            cin >> size;
            if (size <= 0) {
                cout << "Error: Size must be greater than 0!" << endl;
                break;
            }
            if (created) {
                delete[] s.arr;
            }
            s = createStack(size);
            created = true;
            cout << "Stack created with capacity " << size << endl;
            break;
        }

        case 2: {
            if (!created) {
                cout << "Error: Please create a stack first (option 1)!" << endl;
                break;
            }
            cout << "Enter element (number or operator + - * / ^ ( ) ): ";
            Element e = readElement();
            if (e.op == '?') {
                break;
            }
            push(s, e);
            cout << "Pushed: ";
            printEl(e);
            cout << endl;
            break;
        }

        case 3: {
            if (!created) {
                cout << "Error: Please create a stack first (option 1)!" << endl;
                break;
            }
            if (isEmpty(s)) {
                cout << "Error: Stack is empty, nothing to pop!" << endl;
                break;
            }
            Element e = pop(s);
            cout << "Popped: ";
            printEl(e);
            cout << endl;
            break;
        }

        case 4: {
            if (!created) {
                cout << "Error: Please create a stack first (option 1)!" << endl;
                break;
            }
            if (isEmpty(s)) {
                cout << "Error: Stack is empty!" << endl;
                break;
            }

            cout << "\n--- Infix to Postfix Conversion + Evaluation ---" << endl;
            out << "\n--- Infix to Postfix Conversion + Evaluation ---" << endl;

            cout << "Infix: ";
            out << "Infix: ";
            for (int i = 0; i <= s.top; i++) {
                printEl(s.arr[i]);
                cout << " ";
                writeEl(s.arr[i], out);
                out << " ";
            }
            cout << endl;
            out << endl;

            infixToPostfix(s, out);
            break;
        }

        case 5: {
            if (!created) {
                cout << "Error: Please create a stack first (option 1)!" << endl;
                break;
            }
            if (isEmpty(s)) {
                cout << "Error: Stack is empty!" << endl;
                break;
            }

            cout << "\n--- Evaluate Postfix (Single Digit Numbers) ---" << endl;
            out << "\n--- Evaluate Postfix (Single Digit Numbers) ---" << endl;

            cout << "Postfix: ";
            out << "Postfix: ";
            for (int i = 0; i <= s.top; i++) {
                printEl(s.arr[i]);
                cout << " ";
                writeEl(s.arr[i], out);
                out << " ";
            }
            cout << endl;
            out << endl;

            evalPostfix(s, out);
            break;
        }

        case 6: {
            if (!created) {
                cout << "Error: Please create a stack first (option 1)!" << endl;
                break;
            }
            if (isEmpty(s)) {
                cout << "Error: Stack is empty!" << endl;
                break;
            }

            cout << "\n--- Evaluate Postfix (Multi Digit Numbers) ---" << endl;
            out << "\n--- Evaluate Postfix (Multi Digit Numbers) ---" << endl;

            cout << "Postfix: ";
            out << "Postfix: ";
            for (int i = 0; i <= s.top; i++) {
                printEl(s.arr[i]);
                cout << " ";
                writeEl(s.arr[i], out);
                out << " ";
            }
            cout << endl;
            out << endl;

            evalPostfix(s, out);
            break;
        }

        case 7:
            cout << "Goodbye!" << endl;
            break;

        default:
            cout << "Error: Invalid choice! Please enter 1-7." << endl;
        }

    } while (choice != 7);

    out.close();
    if (created) {
        delete[] s.arr;
    }

    return 0;
}

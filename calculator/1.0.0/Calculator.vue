<!-- @app {"title":"Calculator","icon":"icons/apps/org.gnome.Calculator.svg","width":320,"height":480,"singleWindow":true} -->
<template>
    <div class="calculator-app h-full flex flex-col p-4 text-foreground">
        <!-- Display -->
        <div class="mb-4 p-4 bg-muted/50 rounded-lg shadow">
            <div
                class="text-right text-sm text-muted-foreground h-6 truncate"
                :title="expression"
            >
                {{ expression || " " }}
            </div>
            <div
                class="text-right text-3xl font-bold h-10 truncate"
                :title="display"
            >
                {{ display }}
            </div>
        </div>

        <!-- Buttons -->
        <div class="flex-1 grid grid-cols-4 gap-2">
            <!-- Row 1 -->
            <Button
                @click="clear"
                variant="destructive"
                class="col-span-2 h-full text-lg"
            >
                C
            </Button>
            <Button
                @click="backspace"
                variant="secondary"
                class="h-full text-lg"
            >
                ⌫
            </Button>
            <Button
                @click="appendOperator('/')"
                variant="outline"
                class="operator-btn h-full text-lg"
            >
                ÷
            </Button>

            <!-- Row 2 -->
            <Button
                @click="appendNumber('7')"
                variant="outline"
                class="h-full text-xl"
                >7</Button
            >
            <Button
                @click="appendNumber('8')"
                variant="outline"
                class="h-full text-xl"
                >8</Button
            >
            <Button
                @click="appendNumber('9')"
                variant="outline"
                class="h-full text-xl"
                >9</Button
            >
            <Button
                @click="appendOperator('*')"
                variant="outline"
                class="operator-btn h-full text-lg"
            >
                ×
            </Button>

            <!-- Row 3 -->
            <Button
                @click="appendNumber('4')"
                variant="outline"
                class="h-full text-xl"
                >4</Button
            >
            <Button
                @click="appendNumber('5')"
                variant="outline"
                class="h-full text-xl"
                >5</Button
            >
            <Button
                @click="appendNumber('6')"
                variant="outline"
                class="h-full text-xl"
                >6</Button
            >
            <Button
                @click="appendOperator('-')"
                variant="outline"
                class="operator-btn h-full text-lg"
            >
                −
            </Button>

            <!-- Row 4 -->
            <Button
                @click="appendNumber('1')"
                variant="outline"
                class="h-full text-xl"
                >1</Button
            >
            <Button
                @click="appendNumber('2')"
                variant="outline"
                class="h-full text-xl"
                >2</Button
            >
            <Button
                @click="appendNumber('3')"
                variant="outline"
                class="h-full text-xl"
                >3</Button
            >
            <Button
                @click="appendOperator('+')"
                variant="outline"
                class="operator-btn h-full text-lg"
            >
                +
            </Button>

            <!-- Row 5 -->
            <Button
                @click="appendNumber('0')"
                variant="outline"
                class="col-span-2 h-full text-xl"
                >0</Button
            >
            <Button
                @click="appendDecimal"
                variant="outline"
                class="h-full text-xl"
                >.</Button
            >
            <Button
                @click="calculate"
                variant="default"
                class="equals-btn h-full text-lg"
            >
                =
            </Button>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";

// State
const display = ref("0");
const expression = ref("");
const currentNumber = ref("");
const previousNumber = ref("");
const operation = ref(null);
const isNewNumber = ref(true);
const justCalculated = ref(false); // Flag to handle behavior after '='

// Methods
const appendNumber = (num) => {
    if (justCalculated.value) {
        clear(); // Start fresh if a number is pressed after '='
        justCalculated.value = false;
    }
    if (isNewNumber.value) {
        currentNumber.value = num;
        isNewNumber.value = false;
    } else {
        // Prevent multiple leading zeros, but allow '0.'
        if (currentNumber.value === "0" && num === "0") return;
        if (currentNumber.value === "0" && num !== ".")
            currentNumber.value = ""; // remove leading zero if not followed by decimal
        currentNumber.value += num;
    }
    // Limit display length
    if (currentNumber.value.length > 15) {
        currentNumber.value = currentNumber.value.substring(0, 15);
    }
    display.value = currentNumber.value;
    updateExpression();
};

const appendDecimal = () => {
    if (justCalculated.value) {
        clear();
        justCalculated.value = false;
    }
    if (isNewNumber.value) {
        currentNumber.value = "0.";
        isNewNumber.value = false;
    } else if (!currentNumber.value.includes(".")) {
        currentNumber.value += ".";
    }
    if (currentNumber.value.length > 15) {
        currentNumber.value = currentNumber.value.substring(0, 15);
    }
    display.value = currentNumber.value;
    updateExpression();
};

const appendOperator = (op) => {
    justCalculated.value = false;
    // If there's an existing operation and we haven't just started a new number, calculate first
    if (operation.value && !isNewNumber.value && previousNumber.value) {
        calculate(false); // Calculate but don't set justCalculated flag
    }

    // If currentNumber is empty (e.g. after clear or initial load), and an operator is pressed,
    // assume 0 as the first operand.
    if (!currentNumber.value && !previousNumber.value) {
        previousNumber.value = "0";
    } else if (currentNumber.value) {
        previousNumber.value = currentNumber.value;
    }

    operation.value = op;
    isNewNumber.value = true; // Expecting a new number next
    updateExpression();
};

const calculate = (isFinalCalculation = true) => {
    if (
        !operation.value ||
        !previousNumber.value ||
        !currentNumber.value ||
        (isNewNumber.value && isFinalCalculation)
    ) {
        if (
            isFinalCalculation &&
            operation.value &&
            previousNumber.value &&
            !currentNumber.value
        ) {
            currentNumber.value = previousNumber.value;
        } else if (isNewNumber.value && isFinalCalculation) {
            return;
        } else if (
            !operation.value ||
            !previousNumber.value ||
            !currentNumber.value
        ) {
            return;
        }
    }

    const prev = parseFloat(previousNumber.value);
    const curr = parseFloat(currentNumber.value);
    let result = 0;

    switch (operation.value) {
        case "+":
            result = prev + curr;
            break;
        case "-":
            result = prev - curr;
            break;
        case "*":
            result = prev * curr;
            break;
        case "/":
            if (curr === 0) {
                display.value = "Error";
                expression.value = "Cannot divide by zero";
                currentNumber.value = "";
                previousNumber.value = "";
                operation.value = null;
                isNewNumber.value = true;
                justCalculated.value = true;
                return;
            }
            result = prev / curr;
            break;
        default:
            return;
    }

    let resultStr = result.toString();
    if (resultStr.length > 15 && resultStr.includes(".")) {
        const [integerPart, decimalPart] = resultStr.split(".");
        if (integerPart.length > 15) {
            resultStr = result.toExponential(9);
        } else {
            const availableDecimalPlaces = 15 - integerPart.length - 1;
            resultStr = result.toFixed(Math.max(0, availableDecimalPlaces));
        }
    } else if (resultStr.length > 15) {
        resultStr = result.toExponential(9);
    }

    display.value = resultStr;
    if (isFinalCalculation) {
        expression.value = `${previousNumber.value} ${operation.value} ${currentNumber.value} =`;
        previousNumber.value = "";
        currentNumber.value = resultStr;
        operation.value = null;
        isNewNumber.value = true;
        justCalculated.value = true;
    } else {
        expression.value = `${resultStr} ${operation.value}`;
        previousNumber.value = resultStr;
        currentNumber.value = "";
        isNewNumber.value = true;
    }
};

const clear = () => {
    display.value = "0";
    expression.value = "";
    currentNumber.value = "";
    previousNumber.value = "";
    operation.value = null;
    isNewNumber.value = true;
    justCalculated.value = false;
};

const backspace = () => {
    if (justCalculated.value) return;

    if (!isNewNumber.value && currentNumber.value.length > 0) {
        currentNumber.value = currentNumber.value.slice(0, -1);
        if (currentNumber.value === "" || currentNumber.value === "-") {
            currentNumber.value = "0";
            isNewNumber.value = true;
        }
        display.value = currentNumber.value;
    } else if (isNewNumber.value && operation.value && previousNumber.value) {
        operation.value = null;
        currentNumber.value = previousNumber.value;
        previousNumber.value = "";
        isNewNumber.value = false;
        display.value = currentNumber.value;
    }
    updateExpression();
};

const updateExpression = () => {
    if (justCalculated.value && expression.value.endsWith("=")) {
        return;
    }
    let tempExpression = previousNumber.value || "";
    if (operation.value) {
        tempExpression += ` ${operation.value} `;
    }
    if (!isNewNumber.value && currentNumber.value) {
        tempExpression += currentNumber.value;
    } else if (isNewNumber.value && currentNumber.value && !operation.value) {
        tempExpression = currentNumber.value;
    }

    expression.value = tempExpression || " ";
};
</script>

<style scoped>
.calculator-app {
    user-select: none;
}

.operator-btn {
    @apply bg-accent text-accent-foreground hover:bg-accent/90;
}
.equals-btn {
    @apply bg-primary text-primary-foreground hover:bg-primary/90;
}

.grid > * {
    min-height: 0;
}
</style>

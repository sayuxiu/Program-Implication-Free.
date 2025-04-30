#include <stdio.h>
#include <ctype.h>
#include <string.h>
#include <stdlib.h>

// Function to check if a character is an operator
int is_operator(char c) {
    return (c == '+' || c == '*' || c == '-');
}

// Function to check if it is a single atom or a negation
int is_single_valid_formula(char *formula, int start, int end) {
    if (start == end && isalpha(formula[start])) return 1;
    if (formula[start] == '(' && formula[start + 1] == '~' && isalpha(formula[start + 2])
        && formula[start + 3] == ')' && start + 3 == end) return 1;
    return 0;
}

int validate_formula(char *formula) {
    int len = strlen(formula);
    int paren_count = 0;
    int i = 0;
    int last_was_operator = 0;
    int last_was_open_paren = 0;
    int last_was_close_paren = 0;
    int last_was_atom = 0;
    int atom_count = 0;

    if (is_single_valid_formula(formula, 0, len - 1)) return 1;

    if (len == 1 || formula[0] != '(' || formula[len - 1] != ')') return 0;

    if (len == 3 && formula[0] == '(' && isalpha(formula[1]) && formula[2] == ')') return 0;

    int j = 0, nested_paren = 0;
    if (formula[j] == '(') {
        j++;
        while (formula[j] == '(') { nested_paren++; j++; }
        int sub_end = len - 2;
        while (formula[sub_end] == ')') { nested_paren--; sub_end--; }
        if (nested_paren == 0 && is_single_valid_formula(formula, j, sub_end)) return 0;
    }

    while (i < len) {
        char c = formula[i];

        if (isspace(c)) { i++; continue; }

        if (c == '(') {
            paren_count++;
            last_was_open_paren = 1;
            last_was_operator = 0;
            last_was_close_paren = 0;
            last_was_atom = 0;
            atom_count = 0;
        }
        else if (c == ')') {
            paren_count--;
            if (paren_count < 0) return 0;
            if (last_was_operator || last_was_open_paren) return 0;
            if (atom_count != 2) return 0;
            last_was_close_paren = 1;
            last_was_operator = 0;
            last_was_open_paren = 0;
        }
        else if (isalpha(c)) {
            if (last_was_close_paren) return 0;
            if (last_was_atom) return 0;
            last_was_atom = 1;
            last_was_open_paren = 0;
            last_was_operator = 0;
            atom_count++;
        }
        else if (c == '~') {
            if (last_was_atom || last_was_close_paren) return 0;
            if (i + 1 >= len || formula[i + 1] != '(') return 0;
            int j = i + 2;
            if (!isalpha(formula[j])) return 0;
            if (formula[j + 1] != ')') return 0;
            i += 2;
            atom_count++;
        }
        else if (is_operator(c)) {
            if (last_was_operator || last_was_open_paren) return 0;
            if (c == '-' && (i == 0 || i == len - 1)) return 0;
            if (c == '-' && (!last_was_atom && !last_was_close_paren)) return 0;
            last_was_operator = 1;
            last_was_atom = 0;
            last_was_close_paren = 0;
        }
        else {
            return 0;
        }
        i++;
    }
    return (paren_count == 0);
}

// Recursive function to remove implications
void remove_implications(char *formula, char *result) {
    int len = strlen(formula);
    int i = 0, j = 0, k=0;

    while (i < len) {
        if (formula[i] == '(') {
            int start = i;
            int paren_count = 1;
            i++;
            while (i < len && paren_count > 0) {
                if (formula[i] == '(') paren_count++;
                else if (formula[i] == ')') paren_count--;
                i++;
            }
            int end = i;
            char subexpr[500];
            strncpy(subexpr, &formula[start], end - start);
            subexpr[end - start] = '\0';

            // Process the subexpression recursively
            if (strchr(subexpr, '-') != NULL) {
                int p = 1;
                int nested = 0;
                while (subexpr[p] != '\0') {
                    if (subexpr[p] == '(') nested++;
                    else if (subexpr[p] == ')') nested--;
                    else if (subexpr[p] == '-' && nested == 0) {
                        break;
                    }
                    p++;
                }
                if (subexpr[p] == '-') {
                    char left[250], right[250], temp[500];
                    strncpy(left, &subexpr[1], p - 1);
                    left[p - 1] = '\0';
                    strncpy(right, &subexpr[p + 1], strlen(subexpr) - p - 2);
                    right[strlen(subexpr) - p - 2] = '\0';

                    char new_left[500], new_right[500];
                    remove_implications(left, new_left);
                    remove_implications(right, new_right);

                    sprintf(temp, "((~%s)+%s)", new_left, new_right);
                    for (k = 0; temp[k] != '\0'; k++) {
                        result[j++] = temp[k];
                    }
                } else {
                    char temp[500];
                    remove_implications(subexpr, temp);
                    for (k = 0; temp[k] != '\0'; k++) {
                        result[j++] = temp[k];
                    }
                }
            } else {
                for (k = 0; subexpr[k] != '\0'; k++) {
                    result[j++] = subexpr[k];
                }
            }
        } else {
            result[j++] = formula[i++];
        }
    }
    result[j] = '\0';
}

int main() {
    char formula[500];
    printf("Enter a logical proposition: ");
    fgets(formula, sizeof(formula), stdin);
    formula[strcspn(formula, "\n")] = 0;

    if (validate_formula(formula)) {
        printf("Valid logical proposition!\n");

        char no_implications[1000];
        remove_implications(formula, no_implications);
        printf("Implication-free form: %s\n", no_implications);

    } else {
        printf("Invalid logical proposition!\n");
    }

    return 0;
}

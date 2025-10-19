# Лабораторная работа #4 


## Тема: Продвинутая работа с функциями в C



### Цель работы
1. Закрепить навыки работы с функциями, параметрами и возвратом сложных объектов.
2. Освоить передачу параметров и возврат объектов по значению и по адресу.
3. Использовать классические алгоритмы и структуры данных.
4. Научиться работать с современными инструментами сборки проектов (Make, Meson, CMake).
5. Познакомиться с использованием сторонних библиотек.

# Задача 1: Операции с матрицами
## постановка задачи
<img width="473" height="100" alt="image" src="https://github.com/user-attachments/assets/2b3fedae-a6a0-4193-a959-0db7dfa48316" />

# математическая модель
A ∈ ℝ^(m×n) - матрица размером m×n над полем действительных чисел

A = [a_ij], где i = 1,...,m; j = 1,...,n

T: ℝ^(m×n) → ℝ^(n×m)

A^T = [a_ji], где (A^T)_ij = a_ji

+: ℝ^(m×n) × ℝ^(m×n) → ℝ^(m×n)

(A + B)_ij = a_ij + b_ij

×: ℝ^(m×n) × ℝ^(n×p) → ℝ^(m×p)

(AB)_ij = Σ_(k=1)^n a_ik × b_kj

# индетификаторы



# код

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    double **data;
    int rows;
    int cols;
} Matrix;

Matrix* create_matrix(int rows, int cols) {
    Matrix *matrix = (Matrix*)malloc(sizeof(Matrix));
    matrix->rows = rows;
    matrix->cols = cols;
    matrix->data = (double**)malloc(rows * sizeof(double*));
    
    for (int i = 0; i < rows; i++) {
        matrix->data[i] = (double*)malloc(cols * sizeof(double));
        for (int j = 0; j < cols; j++) {
            matrix->data[i][j] = 0.0;
        }
    }
    return matrix;
}

void free_matrix(Matrix *matrix) {
    for (int i = 0; i < matrix->rows; i++) {
        free(matrix->data[i]);
    }
    free(matrix->data);
    free(matrix);
}

void print_matrix(const Matrix *matrix) {
    for (int i = 0; i < matrix->rows; i++) {
        for (int j = 0; j < matrix->cols; j++) {
            printf("%6.2f ", matrix->data[i][j]);
        }
        printf("\n");
    }
}

Matrix* transpose_matrix(const Matrix *matrix) {
    Matrix *result = create_matrix(matrix->cols, matrix->rows);
    
    for (int i = 0; i < matrix->rows; i++) {
        for (int j = 0; j < matrix->cols; j++) {
            result->data[j][i] = matrix->data[i][j];
        }
    }
    return result;
}

Matrix* multiply_matrices(const Matrix *a, const Matrix *b) {
    if (a->cols != b->rows) return NULL;
    
    Matrix *result = create_matrix(a->rows, b->cols);
    
    for (int i = 0; i < a->rows; i++) {
        for (int j = 0; j < b->cols; j++) {
            double sum = 0.0;
            for (int k = 0; k < a->cols; k++) {
                sum += a->data[i][k] * b->data[k][j];
            }
            result->data[i][j] = sum;
        }
    }
    return result;
}

int main() {
    Matrix *A = create_matrix(2, 3);
    Matrix *B = create_matrix(3, 2);
    
    A->data[0][0] = 1; A->data[0][1] = 2; A->data[0][2] = 3;
    A->data[1][0] = 4; A->data[1][1] = 5; A->data[1][2] = 6;
    
    B->data[0][0] = 7; B->data[0][1] = 8;
    B->data[1][0] = 9; B->data[1][1] = 10;
    B->data[2][0] = 11; B->data[2][1] = 12;
    
    printf("Матрица A:\n");
    print_matrix(A);
    
    printf("\nМатрица B:\n");
    print_matrix(B);
    
    Matrix *C = multiply_matrices(A, B);
    printf("\nУмножение A * B:\n");
    print_matrix(C);
    
    free_matrix(A);
    free_matrix(B);
    free_matrix(C);
    
    return 0;
}
```
# результат 
<img width="378" height="219" alt="image" src="https://github.com/user-attachments/assets/bd70f768-07c1-44ea-98b3-ef22a5fd528c" />

<img width="252" height="119" alt="image" src="https://github.com/user-attachments/assets/58124765-c7fd-4018-ba40-72283b6b35f0" />


# Задача 2: Упрощённый парсер JSON

## постановка задачи
<img width="731" height="131" alt="image" src="https://github.com/user-attachments/assets/b140aab5-4718-40e1-8ce6-8920a4ac330c" />


## математическая модель


JSON → { pairs }

pairs → ε | pair | pair, pairs  

pair → string : value

value → string | number | boolean | null

JSON = { (key, value) }

где key ∈ String, value ∈ Value

Value = String ∪ Number ∪ Boolean ∪ {null}

parse: String → JsonData

JsonData = { (k₁, v₁), (k₂, v₂), ..., (kₙ, vₙ) }



# индетификаторы
| Идентификатор | Тип | Назначение |
|---------------|-----|------------|
| `JsonData` | `struct` | Хранит распарсенные данные |
| `keys` | `char[100][100]` | Массив ключей |
| `values` | `char[100][100]` | Массив значений |
| `count` | `int` | Количество пар ключ-значение |
| `parse_json` | Парсит JSON строку |
| `print_json_data` | Выводит результат |
| `json_string` | Входная JSON строка |
| `json` | cJSON объект |
| `item` | Текущий элемент JSON |
| `cJSON_Parse` | Парсит JSON строку |
| `cJSON_Delete` | Освобождает память |
| `cJSON_IsString` | Проверяет тип string |
| `cJSON_IsNumber` | Проверяет тип number |
| `cJSON_IsBool` | Проверяет тип boolean |



# Код
```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

typedef struct {
    char keys[10][50];
    char values[10][50];
    int count;
} JsonData;

JsonData parse_simple_json(const char *json_str) {
    JsonData result = {0};
    char buffer[500];
    strcpy(buffer, json_str);
    
    // Убираем пробелы и {}
    char *clean = buffer;
    while (*clean) {
        if (*clean == ' ' || *clean == '\t' || *clean == '\n') {
            memmove(clean, clean + 1, strlen(clean));
        } else {
            clean++;
        }
    }
    
    if (buffer[0] == '{') memmove(buffer, buffer + 1, strlen(buffer));
    if (buffer[strlen(buffer)-1] == '}') buffer[strlen(buffer)-1] = '\0';
    
    char *token = strtok(buffer, ",");
    while (token && result.count < 10) {
        char *colon = strchr(token, ':');
        if (colon) {
            *colon = '\0';
            char *key = token;
            char *value = colon + 1;
            
            // Убираем кавычки
            if (key[0] == '"') memmove(key, key + 1, strlen(key));
            if (key[strlen(key)-1] == '"') key[strlen(key)-1] = '\0';
            if (value[0] == '"') memmove(value, value + 1, strlen(value));
            if (value[strlen(value)-1] == '"') value[strlen(value)-1] = '\0';
            
            strcpy(result.keys[result.count], key);
            strcpy(result.values[result.count], value);
            result.count++;
        }
        token = strtok(NULL, ",");
    }
    
    return result;
}

void print_json_data(const JsonData *data) {
    printf("Parsed JSON (%d items):\n", data->count);
    for (int i = 0; i < data->count; i++) {
        printf("  %s: %s\n", data->keys[i], data->values[i]);
    }
}

int main() {
    const char *json = "{\"name\": \"John\", \"age\": \"30\", \"city\": \"Moscow\"}";
    printf("JSON: %s\n", json);
    
    JsonData data = parse_simple_json(json);
    print_json_data(&data);
    
    return 0;
}
```

# результат
<img width="515" height="133" alt="image" src="https://github.com/user-attachments/assets/b852db70-e7d6-4fda-b0cd-7f40cc2bf4a2" />




# Задача 3: Решение систем линейных уравнений

## постановка задачи
<img width="568" height="106" alt="image" src="https://github.com/user-attachments/assets/3f8a480f-16d2-45a0-96ab-6c3176989d49" />


# математическая модель


В матричной форме:
**A × X = B**, где:
- **A** - матрица коэффициентов n×n
- **X** - вектор неизвестных [x₁, x₂, ..., xₙ]ᵀ
- **B** - вектор свободных членов [b₁, b₂, ..., bₙ]ᵀ

## Алгоритм метода Гаусса

### 1. Прямой ход
Приведение к треугольному виду:

# индетификаторы
| Идентификатор | Тип | Назначение |
|---------------|-----|------------|
| `A` | `double**` | Матрица коэффициентов системы |
| `B` | `double*` | Вектор свободных членов |
| `solution` | `double*` | Вектор решений системы |
| `matrix` | `double**` | Копия матрицы для вычислений |
| `vector` | `double*` | Копия вектора для вычислений |
|---------------|-----------|------------|------------|
| `gauss_elimination` | `A`, `B`, `n` | `double*` | Решение системы методом Гаусса |
| `create_matrix` | `rows`, `cols` | `double**` | Создание матрицы |
| `create_vector` | `size` | `double*` | Создание вектора |
| `free_matrix` | `matrix`, `rows` | `void` | Освобождение памяти матрицы |
| `free_vector` | `vector` | `void` | Освобождение памяти вектора |
| `print_system` | `A`, `B`, `n` | `void` | Вывод системы уравнений |
| `print_solution` | `X`, `n` | `void` | Вывод решения || `n` | `int` | Размерность системы |
| `i, j, k` | `int` | Индексы циклов |
| `factor` | `double` | Множитель для исключения переменной |
| `sum` | `double` | Вспомогательная сумма |
| `A` | **A** | Матрица коэффициентов |
| `B` | **B** | Вектор правых частей |
| `solution` | **X** | Вектор неизвестных |
| `factor` | mᵢₖ | Множитель Гаусса |
| `EPS` | ε | Малая величина для проверки вырожденности |
| `EPS` | `1e-10` | Точность для сравнения с нулем |


# Код
```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

#define EPS 1e-10

double* gauss_elimination(double** A, double* B, int n) {
    double** matrix = (double**)malloc(n * sizeof(double*));
    double* vector = (double*)malloc(n * sizeof(double));
    
    for (int i = 0; i < n; i++) {
        matrix[i] = (double*)malloc(n * sizeof(double));
        for (int j = 0; j < n; j++) {
            matrix[i][j] = A[i][j];
        }
        vector[i] = B[i];
    }
    
    for (int k = 0; k < n; k++) {
        if (fabs(matrix[k][k]) < EPS) {
            printf("Матрица вырождена\n");
            return NULL;
        }
        
        for (int i = k + 1; i < n; i++) {
            double factor = matrix[i][k] / matrix[k][k];
            for (int j = k; j < n; j++) {
                matrix[i][j] -= factor * matrix[k][j];
            }
            vector[i] -= factor * vector[k];
        }
    }
    
    double* solution = (double*)malloc(n * sizeof(double));
    
    for (int i = n - 1; i >= 0; i--) {
        double sum = 0.0;
        for (int j = i + 1; j < n; j++) {
            sum += matrix[i][j] * solution[j];
        }
        solution[i] = (vector[i] - sum) / matrix[i][i];
    }
    
    for (int i = 0; i < n; i++) {
        free(matrix[i]);
    }
    free(matrix);
    free(vector);
    
    return solution;
}

double** create_matrix(int rows, int cols) {
    double** matrix = (double**)malloc(rows * sizeof(double*));
    for (int i = 0; i < rows; i++) {
        matrix[i] = (double*)malloc(cols * sizeof(double));
    }
    return matrix;
}

double* create_vector(int size) {
    return (double*)malloc(size * sizeof(double));
}

void free_matrix(double** matrix, int rows) {
    for (int i = 0; i < rows; i++) {
        free(matrix[i]);
    }
    free(matrix);
}

void free_vector(double* vector) {
    free(vector);
}

void print_system(double** A, double* B, int n) {
    printf("Система уравнений:\n");
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            printf("%6.2fx%d ", A[i][j], j + 1);
            if (j < n - 1) printf("+ ");
        }
        printf("= %6.2f\n", B[i]);
    }
}

void print_solution(double* X, int n) {
    printf("Решение системы:\n");
    for (int i = 0; i < n; i++) {
        printf("x%d = %8.4f\n", i + 1, X[i]);
    }
}

int main() {
    int n = 3;
    
    double** A = create_matrix(n, n);
    double* B = create_vector(n);
    
    A[0][0] = 2; A[0][1] = 1; A[0][2] = -1;
    A[1][0] = -3; A[1][1] = -1; A[1][2] = 2;
    A[2][0] = -2; A[2][1] = 1; A[2][2] = 2;
    
    B[0] = 8;
    B[1] = -11;
    B[2] = -3;
    
    print_system(A, B, n);
    
    double* solution = gauss_elimination(A, B, n);
    
    if (solution != NULL) {
        print_solution(solution, n);
        free_vector(solution);
    }
    
    free_matrix(A, n);
    free_vector(B);
    
    return 0;
}

```

# реэультат
<img width="414" height="196" alt="image" src="https://github.com/user-attachments/assets/7515e7c2-220d-4af8-99f7-66acf3477788" />


# Задача 4: Работа с деревьями

## постановка задачи
<img width="735" height="108" alt="image" src="https://github.com/user-attachments/assets/46e21d75-e5f6-4f9d-93d1-8887b2fdb107" />


# математическая мождель
нету

# индетификаторы
| Идентификатор | Тип | Назначение |
|---------------|-----|------------|
| `TreeNode` | `struct` | Узел бинарного дерева |
| `data` | `int` | Ключ узла |
| `left` | `TreeNode*` | Указатель на левое поддерево |
| `right` | `TreeNode*` | Указатель на правое поддерево |
| `root` | `TreeNode*` | Корень дерева |
| `create_node` | `int data` | `TreeNode*` | Создание нового узла |
| `insert` | `TreeNode**, int` | `void` | Вставка элемента в дерево |
| `search` | `TreeNode*, int` | `TreeNode*` | Поиск элемента в дереве |
| `inorder_traversal` | `TreeNode*` | `void` | Обход L-N-R |
| `preorder_traversal` | `TreeNode*` | `void` | Обход N-L-R |
| `postorder_traversal` | `TreeNode*` | `void` | Обход L-R-N |
| `store_inorder` | `TreeNode*, int*, int*` | `void` | Сохранение элементов в массив |
| `sorted_array_to_bst` | `int*, int, int` | `TreeNode*` | Построение BST из отсортированного массива |
| `balance_tree` | `TreeNode**` | `void` | Балансировка дерева |
| `free_tree` | `TreeNode*` | `void` | Освобождение памяти дерева |
| `arr` | `int*` | Массив для хранения элементов дерева |
| `index` | `int` | Текущий индекс в массиве |
| `start, end` | `int` | Границы подмассива |
| `mid` | `int` | Середина массива |
| `size` | `int` | Размер дерева/массива |
| Идентификатор | Назначение |
|---------------|------------|
| `qsort` | Сортировка массива для балансировки |
| `malloc` | Выделение памяти |
| `free` | Освобождение памяти |

# код
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct TreeNode {
    int data;
    struct TreeNode* left;
    struct TreeNode* right;
} TreeNode;

TreeNode* create_node(int data) {
    TreeNode* new_node = (TreeNode*)malloc(sizeof(TreeNode));
    new_node->data = data;
    new_node->left = NULL;
    new_node->right = NULL;
    return new_node;
}

void insert(TreeNode** root, int data) {
    if (*root == NULL) {
        *root = create_node(data);
        return;
    }
    
    if (data < (*root)->data) {
        insert(&(*root)->left, data);
    } else {
        insert(&(*root)->right, data);
    }
}

TreeNode* search(TreeNode* root, int key) {
    if (root == NULL || root->data == key) {
        return root;
    }
    
    if (key < root->data) {
        return search(root->left, key);
    } else {
        return search(root->right, key);
    }
}

void inorder_traversal(TreeNode* root) {
    if (root != NULL) {
        inorder_traversal(root->left);
        printf("%d ", root->data);
        inorder_traversal(root->right);
    }
}

void preorder_traversal(TreeNode* root) {
    if (root != NULL) {
        printf("%d ", root->data);
        preorder_traversal(root->left);
        preorder_traversal(root->right);
    }
}

void postorder_traversal(TreeNode* root) {
    if (root != NULL) {
        postorder_traversal(root->left);
        postorder_traversal(root->right);
        printf("%d ", root->data);
    }
}

void store_inorder(TreeNode* root, int* arr, int* index) {
    if (root == NULL) return;
    store_inorder(root->left, arr, index);
    arr[(*index)++] = root->data;
    store_inorder(root->right, arr, index);
}

int count_nodes(TreeNode* root) {
    if (root == NULL) return 0;
    return 1 + count_nodes(root->left) + count_nodes(root->right);
}

int compare(const void* a, const void* b) {
    return (*(int*)a - *(int*)b);
}

TreeNode* sorted_array_to_bst(int* arr, int start, int end) {
    if (start > end) return NULL;
    
    int mid = (start + end) / 2;
    TreeNode* root = create_node(arr[mid]);
    
    root->left = sorted_array_to_bst(arr, start, mid - 1);
    root->right = sorted_array_to_bst(arr, mid + 1, end);
    
    return root;
}

void balance_tree(TreeNode** root) {
    int size = count_nodes(*root);
    int* arr = (int*)malloc(size * sizeof(int));
    int index = 0;
    
    store_inorder(*root, arr, &index);
    qsort(arr, size, sizeof(int), compare);
    
    free_tree(*root);
    *root = sorted_array_to_bst(arr, 0, size - 1);
    
    free(arr);
}

void free_tree(TreeNode* root) {
    if (root != NULL) {
        free_tree(root->left);
        free_tree(root->right);
        free(root);
    }
}

int main() {
    TreeNode* root = NULL;
    
    int values[] = {50, 30, 70, 20, 40, 60, 80, 10, 25, 35, 45};
    int n = sizeof(values) / sizeof(values[0]);
    
    for (int i = 0; i < n; i++) {
        insert(&root, values[i]);
    }
    
    printf("Исходное дерево (in-order): ");
    inorder_traversal(root);
    printf("\n");
    
    printf("Pre-order: ");
    preorder_traversal(root);
    printf("\n");
    
    printf("Post-order: ");
    postorder_traversal(root);
    printf("\n");
    
    int search_key = 40;
    TreeNode* found = search(root, search_key);
    if (found != NULL) {
        printf("Найден элемент: %d\n", found->data);
    } else {
        printf("Элемент %d не найден\n", search_key);
    }
    
    balance_tree(&root);
    printf("Сбалансированное дерево (in-order): ");
    inorder_traversal(root);
    printf("\n");
    
    free_tree(root);
    return 0;
}
```
# результат

<img width="703" height="131" alt="image" src="https://github.com/user-attachments/assets/34868dd3-5d50-419c-92dc-dd83fdef1417" />


# Задача 5: Чтение веб-страницы

## постановка задачи

<img width="522" height="122" alt="image" src="https://github.com/user-attachments/assets/85d4ecb2-78e2-459e-9ea4-cadadb559c20" />



# математическая модель

HTTP запрос-ответ модель:

Request: GET URL HTTP/1.1

Headers...

Body...

Response: HTTP/1.1 Status Code

Headers...

Body...

HTTP_Request = {

method: String,

url: URL,

headers: Map<String, String>,

body: String

}

HTTP_Response = {

status_code: Int,

headers: Map<String, String>,

body: String

}


# идентификаторы

| Идентификатор | Тип | Назначение |
|---------------|-----|------------|
| `WebPage` | `struct` | Структура веб-страницы |
| `url` | `char*` | URL запрашиваемой страницы |
| `headers` | `char*` | Заголовки HTTP ответа |
| `body` | `char*` | Тело HTTP ответа |
| `status_code` | `long` | HTTP статус код |
| `curl` | `CURL*` | Указатель на CURL сессию |
| `fetch_webpage` | `const char* url` | `WebPage` | Получение веб-страницы |
| `init_webpage` | `void` | `WebPage` | Инициализация структуры |
| `free_webpage` | `WebPage*` | `void` | Освобождение памяти |
| `write_callback` | `char*, size_t, size_t, void*` | `size_t` | Callback для записи данных |
| `header_callback` | `char*, size_t, size_t, void*` | `size_t` | Callback для заголовков |
| `res` | `CURLcode` | Результат выполнения CURL |
| `response_data` | `struct` | Буфер для данных ответа |
| `data` | `char*` | Указатель на данные |
| `size` | `size_t` | Размер данных |
| `url` | `const char*` | URL для запроса |
| `curl_global_init` | Инициализация CURL |
| `curl_easy_init` | Создание CURL сессии |
| `curl_easy_setopt` | Настройка параметров CURL |
| `curl_easy_perform` | Выполнение HTTP запроса |
| `curl_easy_cleanup` | Очистка CURL сессии |
| `curl_global_cleanup` | Очистка CURL |
| `curl_easy_getinfo` | Получение информации о запросе |

# код
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char* headers;
    char* body;
    long status_code;
    char* url;
} WebPage;

WebPage init_webpage() {
    WebPage wp;
    wp.headers = NULL;
    wp.body = NULL;
    wp.status_code = 0;
    wp.url = NULL;
    return wp;
}

void free_webpage(WebPage* wp) {
    if (wp->headers) free(wp->headers);
    if (wp->body) free(wp->body);
    if (wp->url) free(wp->url);
}


WebPage fetch_webpage(const char* url) {
    WebPage webpage = init_webpage();
    webpage.url = strdup(url);
    webpage.status_code = 200;
    
    // Тестовые заголовки
    webpage.headers = strdup(
        "HTTP/1.1 200 OK\r\n"
        "Content-Type: text/html\r\n"
        "Server: TestServer\r\n"
        "Connection: close\r\n"
    );
    
    
    webpage.body = strdup(
        "<!DOCTYPE html>\n"
        "<html>\n"
        "<head>\n"
        "    <title>Test Page</title>\n"
        "</head>\n"
        "<body>\n"
        "    <h1>Hello World!</h1>\n"
        "    <p>This is a simulated web page for lab work.</p>\n"
        "    <p>URL: "
    );
    
    
    char* new_body = realloc(webpage.body, strlen(webpage.body) + strlen(url) + 100);
    if (new_body) {
        webpage.body = new_body;
        strcat(webpage.body, url);
        strcat(webpage.body, "</p>\n</body>\n</html>");
    }
    
    return webpage;
}

void print_webpage_info(const WebPage* wp) {
    printf("URL: %s\n", wp->url);
    printf("Status Code: %ld\n", wp->status_code);
    printf("\nHeaders:\n%s\n", wp->headers);
    printf("Body (%zu bytes):\n%s\n", strlen(wp->body), wp->body);
}

int main() {
    printf("=== HTTP Client (Simulation) ===\n\n");
    
    WebPage page = fetch_webpage("http://example.com/test");
    
    if (page.body != NULL) {
        print_webpage_info(&page);
        free_webpage(&page);
    } else {
        printf("Не удалось загрузить страницу\n");
    }
    
    
    printf("\n=== Second Test ===\n");
    WebPage page2 = fetch_webpage("http://localhost:8080/api/data");
    print_webpage_info(&page2);
    free_webpage(&page2);
    
    return 0;
}

```

# результат
=== HTTP Client (Simulation) ===

URL: http://example.com/test
Status Code: 200

Headers:
HTTP/1.1 200 OK
Content-Type: text/html
Server: TestServer
Connection: close

Body (209 bytes):
<!DOCTYPE html>
<html>
<head>
    <title>Test Page</title>
</head>
<body>
    <h1>Hello World!</h1>
    <p>This is a simulated web page for lab work.</p>
    <p>URL: http://example.com/test</p>
</body>
</html>

=== Second Test ===
URL: http://localhost:8080/api/data
Status Code: 200

Headers:
HTTP/1.1 200 OK
Content-Type: text/html
Server: TestServer
Connection: close

Body (216 bytes):
<!DOCTYPE html>
<html>
<head>
    <title>Test Page</title>
</head>
<body>
    <h1>Hello World!</h1>
    <p>This is a simulated web page for lab work.</p>
    <p>URL: http://localhost:8080/api/data</p>
</body>
</html>



# Задача 6: Сортировка и поиск в динамическом массиве

## постановка задачи
<img width="761" height="103" alt="image" src="https://github.com/user-attachments/assets/0bcf71dd-44d7-43f4-85c0-61f2de6aaae2" />

# математическая модель

Для массива A размера n и элемента x:

- **Сортировка**: найти перестановку π такую, что A[π(1)] ≤ A[π(2)] ≤ ... ≤ A[π(n)]
- 
- **Бинарный поиск**: найти индекс i такой, что A[i] = x, используя свойство отсортированности
- function binary_search(A, left, right, x):

while left ≤ right:

mid = ⌊(left + right) / 2⌋

if A[mid] == x:

return mid

else if A[mid] < x:

left = mid + 1

else:

right = mid - 1

return -1

# индентификаторы

| Идентификатор | Тип | Назначение |
|---------------|-----|------------|
| `Person` | `struct` | Структура человека |
| `name` | `char[50]` | Имя |
| `age` | `int` | Возраст |
| `gender` | `enum` | Пол |
| `Gender` | `enum` | Перечисление пола |
| `people` | `Person*` | Массив людей |
|---------------|------------|
| `create_array` | Создает массив |
| `free_array` | Освобождает память |
| `sort_by_name` | Сортирует по имени |
| `sort_by_age` | Сортирует по возрасту |
| `binary_search_by_name` | Ищет по имени |
| `binary_search_by_age` | Ищет по возрасту |
| `print_person` | Выводит данные человека |
| `print_array` | Выводит массив |
| `compare_by_name` | Сравнивает по имени (для qsort) |
| `compare_by_age` | Сравнивает по возрасту (для qsort) |
| `gender_to_string` | Конвертирует пол в строку |
|---------------|-----|------------|
| `size` | `int` | Размер массива |
| `left`, `right`, `mid` | `int` | Границы поиска |
| `target_name` | `const char*` | Искомое имя |
| `target_age` | `int` | Искомый возраст |
| `found` | `Person*` | Найденный элемент |
| `qsort` | Сортировка |
| `strcmp` | Сравнение строк |
| `malloc`, `free` | Работа с памятью |


# код

```c

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef enum {
    MALE,
    FEMALE,
    OTHER
} Gender;

typedef struct {
    char name[50];
    int age;
    Gender gender;
} Person;

int compare_by_name(const void* a, const void* b) {
    const Person* p1 = (const Person*)a;
    const Person* p2 = (const Person*)b;
    return strcmp(p1->name, p2->name);
}

int compare_by_age(const void* a, const void* b) {
    const Person* p1 = (const Person*)a;
    const Person* p2 = (const Person*)b;
    return p1->age - p2->age;
}

Person* create_array(int size) {
    return (Person*)malloc(size * sizeof(Person));
}

void free_array(Person* arr) {
    free(arr);
}

void sort_by_name(Person* arr, int size) {
    qsort(arr, size, sizeof(Person), compare_by_name);
}

void sort_by_age(Person* arr, int size) {
    qsort(arr, size, sizeof(Person), compare_by_age);
}

Person* binary_search_by_name(Person* arr, int size, const char* target_name) {
    int left = 0;
    int right = size - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        int cmp = strcmp(arr[mid].name, target_name);
        
        if (cmp == 0) {
            return &arr[mid];
        } else if (cmp < 0) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return NULL;
}

Person* binary_search_by_age(Person* arr, int size, int target_age) {
    int left = 0;
    int right = size - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid].age == target_age) {
            return &arr[mid];
        } else if (arr[mid].age < target_age) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return NULL;
}

const char* gender_to_string(Gender gender) {
    switch (gender) {
        case MALE: return "Male";
        case FEMALE: return "Female";
        case OTHER: return "Other";
        default: return "Unknown";
    }
}

void print_person(const Person* person) {
    if (person) {
        printf("Name: %s, Age: %d, Gender: %s\n", 
               person->name, person->age, gender_to_string(person->gender));
    }
}

void print_array(Person* arr, int size) {
    for (int i = 0; i < size; i++) {
        printf("%d: ", i + 1);
        print_person(&arr[i]);
    }
}

int main() {
    int size = 5;
    Person* people = create_array(size);
    
    // Инициализация данных
    strcpy(people[0].name, "Alice"); people[0].age = 25; people[0].gender = FEMALE;
    strcpy(people[1].name, "Bob"); people[1].age = 30; people[1].gender = MALE;
    strcpy(people[2].name, "Charlie"); people[2].age = 22; people[2].gender = MALE;
    strcpy(people[3].name, "Diana"); people[3].age = 28; people[3].gender = FEMALE;
    strcpy(people[4].name, "Eve"); people[4].age = 35; people[4].gender = FEMALE;
    
    printf("Original array:\n");
    print_array(people, size);
    
    // Сортировка и поиск по имени
    printf("\nSorted by name:\n");
    sort_by_name(people, size);
    print_array(people, size);
    
    Person* found = binary_search_by_name(people, size, "Charlie");
    printf("\nSearch for 'Charlie':\n");
    print_person(found);
    
    // Сортировка и поиск по возрасту
    printf("\nSorted by age:\n");
    sort_by_age(people, size);
    print_array(people, size);
    
    found = binary_search_by_age(people, size, 28);
    printf("\nSearch for age 28:\n");
    print_person(found);
    
    // Поиск несуществующего элемента
    found = binary_search_by_name(people, size, "Zack");
    printf("\nSearch for 'Zack':\n");
    if (!found) {
        printf("Not found\n");
    }
    
    free_array(people);
    return 0;
}
```

# результат
<img width="380" height="146" alt="image" src="https://github.com/user-attachments/assets/cbee60c6-1643-4ec4-bb1f-f297185fd329" />

<img width="383" height="164" alt="image" src="https://github.com/user-attachments/assets/1b45827e-7ac5-4341-8686-8def33a47faa" />

<img width="362" height="221" alt="image" src="https://github.com/user-attachments/assets/cbb3ff47-daed-4446-8347-7b70bb4cc973" />


<img width="359" height="149" alt="image" src="https://github.com/user-attachments/assets/453987b6-fd93-4dd4-b512-13647ec98801" />




# Задача 7: Кэширование вычислений


## постановка задачи

<img width="611" height="95" alt="image" src="https://github.com/user-attachments/assets/4f2dfd4c-bb42-4c5a-a6aa-4f64287f1377" />


# математичекая модель

Рекуррентная формула:

F(0) = 0

F(1) = 1

F(n) = F(n-1) + F(n-2) для n ≥ 2


Кэширование результатов:

cache[n] = F(n)


# индетификаторы

| Идентификатор | Тип | Назначение |
|---------------|-----|------------|
| `CacheEntry` | `struct` | Элемент кэша |
| `n` | `int` | Число Фибоначчи |
| `value` | `long long` | Значение F(n) |
| `fib_cache` | `CacheEntry*` | Динамический кэш |
| `cache_size` | `int` | Размер кэша |
|---------------|------------|
| `fibonacci` | Вычисляет число Фибоначчи |
| `init_cache` | Инициализирует кэш |
| `expand_cache` | Увеличивает кэш |
| `get_from_cache` | Получает значение из кэша |
| `add_to_cache` | Добавляет в кэш |
| `free_cache` | Освобождает память |
| `print_cache` | Выводит кэш |
|---------------|-----|------------|
| `n` | `int` | Входное число |
| `result` | `long long` | Результат |
| `i` | `int` | Индекс |


# код

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int n;
    long long value;
} CacheEntry;

CacheEntry* fib_cache = NULL;
int cache_size = 0;
int cache_capacity = 0;

void init_cache() {
    cache_capacity = 10;
    fib_cache = (CacheEntry*)malloc(cache_capacity * sizeof(CacheEntry));
    cache_size = 0;
}

void expand_cache() {
    cache_capacity *= 2;
    fib_cache = (CacheEntry*)realloc(fib_cache, cache_capacity * sizeof(CacheEntry));
}

long long get_from_cache(int n) {
    for (int i = 0; i < cache_size; i++) {
        if (fib_cache[i].n == n) {
            return fib_cache[i].value;
        }
    }
    return -1;
}

void add_to_cache(int n, long long value) {
    if (cache_size >= cache_capacity) {
        expand_cache();
    }
    fib_cache[cache_size].n = n;
    fib_cache[cache_size].value = value;
    cache_size++;
}

long long fibonacci(int n) {
    if (n < 0) return -1;
    if (n == 0) return 0;
    if (n == 1) return 1;
    
    long long cached = get_from_cache(n);
    if (cached != -1) {
        return cached;
    }
    
    long long result = fibonacci(n - 1) + fibonacci(n - 2);
    add_to_cache(n, result);
    return result;
}

void free_cache() {
    free(fib_cache);
    fib_cache = NULL;
    cache_size = 0;
    cache_capacity = 0;
}

void print_cache() {
    printf("Cache content (%d entries):\n", cache_size);
    for (int i = 0; i < cache_size; i++) {
        printf("F(%d) = %lld\n", fib_cache[i].n, fib_cache[i].value);
    }
}

int main() {
    init_cache();
    
    printf("Fibonacci numbers with memoization:\n");
    
    int tests[] = {0, 1, 5, 10, 15, 20, 25};
    int num_tests = sizeof(tests) / sizeof(tests[0]);
    
    for (int i = 0; i < num_tests; i++) {
        int n = tests[i];
        long long result = fibonacci(n);
        printf("F(%d) = %lld\n", n, result);
    }
    
    printf("\n");
    print_cache();
    
    free_cache();
    return 0;
}

```

# результат
<img width="354" height="198" alt="image" src="https://github.com/user-attachments/assets/b9df91cc-e52f-42ff-b25c-3bf6794ead86" />


Cache content (24 entries):

F(2) = 1

F(3) = 2

F(4) = 3

F(5) = 5

F(6) = 8

F(7) = 13

F(8) = 21

F(9) = 34

F(10) = 55

F(11) = 89

F(12) = 144

F(13) = 233

F(14) = 377

F(15) = 610

F(16) = 987

F(17) = 1597

F(18) = 2584

F(19) = 4181

F(20) = 6765

F(21) = 10946

F(22) = 17711

F(23) = 28657

F(24) = 46368

F(25) = 75025



# Задача 8: Преобразование текста

## постановка задачи
<img width="704" height="104" alt="image" src="https://github.com/user-attachments/assets/75dd94a6-08d6-4da4-9f08-b226420f937a" />



# математическая модель

Для строки S длины n:

f(c) = count(c ∈ S) / n

где count(c ∈ S) - количество вхождений символа c в S


## Распределение частот

Гистограмма: H = {(c₁, f₁), (c₂, f₂), ..., (cₖ, fₖ)}

где f₁ ≥ f₂ ≥ ... ≥ fₖ


# индетификаторы

| Идентификатор | Тип | Назначение |
|---------------|-----|------------|
| `CharFrequency` | `struct` | Частота символа |
| `character` | `char` | Символ |
| `frequency` | `int` | Частота |
| `count` | `int` | Счетчик |
| `freq_array` | `CharFrequency*` | Массив частот |
| `count_frequencies` | Считает частоты символов |
| `sort_frequencies` | Сортирует по убыванию частоты |
| `print_frequencies` | Выводит частоты |
| `compare_freq` | Сравнивает частоты (для qsort) |
| `init_freq_array` | Инициализирует массив частот |
| `text` | `const char*` | Входной текст |
| `freq_count` | `int` | Количество уникальных символов |
| `i, j` | `int` | Индексы |
| `total_chars` | `int` | Общее количество символов |


# код

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char character;
    int frequency;
} CharFrequency;

int compare_freq(const void* a, const void* b) {
    const CharFrequency* fa = (const CharFrequency*)a;
    const CharFrequency* fb = (const CharFrequency*)b;
    return fb->frequency - fa->frequency; // По убыванию
}

void init_freq_array(CharFrequency* freq_array) {
    for (int i = 0; i < 256; i++) {
        freq_array[i].character = (char)i;
        freq_array[i].frequency = 0;
    }
}

CharFrequency* count_frequencies(const char* text, int* unique_count) {
    CharFrequency* freq_array = (CharFrequency*)malloc(256 * sizeof(CharFrequency));
    init_freq_array(freq_array);
    
    int total_chars = strlen(text);
    *unique_count = 0;
    
    for (int i = 0; i < total_chars; i++) {
        unsigned char c = text[i];
        if (freq_array[c].frequency == 0) {
            (*unique_count)++;
        }
        freq_array[c].frequency++;
    }
    
    return freq_array;
}

CharFrequency* sort_frequencies(CharFrequency* freq_array, int unique_count) {
    CharFrequency* result = (CharFrequency*)malloc(unique_count * sizeof(CharFrequency));
    int index = 0;
    
    for (int i = 0; i < 256; i++) {
        if (freq_array[i].frequency > 0) {
            result[index++] = freq_array[i];
        }
    }
    
    qsort(result, unique_count, sizeof(CharFrequency), compare_freq);
    return result;
}

void print_frequencies(const CharFrequency* freq_array, int count, int total_chars) {
    printf("Character frequencies (total chars: %d):\n", total_chars);
    printf("Char | Count | Frequency\n");
    printf("-----|-------|----------\n");
    
    for (int i = 0; i < count && i < 20; i++) {
        double freq_percent = (double)freq_array[i].frequency / total_chars * 100;
        printf("  %c  | %5d | %6.2f%%\n", 
               freq_array[i].character, 
               freq_array[i].frequency, 
               freq_percent);
    }
}

void print_histogram(const CharFrequency* freq_array, int count) {
    printf("\nHistogram (top 10):\n");
    for (int i = 0; i < count && i < 10; i++) {
        printf("%c: ", freq_array[i].character);
        int bars = (freq_array[i].frequency + 9) / 10;
        for (int j = 0; j < bars; j++) {
            printf("█");
        }
        printf(" %d\n", freq_array[i].frequency);
    }
}

int main() {
    const char* text = "Hello World! This is a sample text for character frequency analysis.";
    
    int unique_count;
    int total_chars = strlen(text);
    
    CharFrequency* freq_array = count_frequencies(text, &unique_count);
    CharFrequency* sorted_freq = sort_frequencies(freq_array, unique_count);
    
    print_frequencies(sorted_freq, unique_count, total_chars);
    print_histogram(sorted_freq, unique_count);
    
    free(freq_array);
    free(sorted_freq);
    
    return 0;
}
```

# результат
<img width="402" height="505" alt="image" src="https://github.com/user-attachments/assets/7914740e-540b-4d2b-9edd-05630ecc5c71" />




# Задача 9: Упаковка и распаковка данных

## постановка задачи


<img width="745" height="107" alt="image" src="https://github.com/user-attachments/assets/5bb4be85-0a09-4b13-9552-729f5b782159" />


# математическая модель

Сериализация: Struct → Byte[]

Десериализация: Byte[] → Struct


Биективное преобразование:

pack: S → B

unpack: B → S

unpack(pack(S)) = S


# индетификаторы

| Идентификатор | Тип | Назначение |
|---------------|-----|------------|
| `Student` | `struct` | Структура студента |
| `id` | `int` | ID студента |
| `name` | `char[50]` | Имя |
| `grade` | `float` | Оценка |
| `students` | `Student*` | Массив студентов |
| `pack_data` | Упаковывает данные в файл |
| `unpack_data` | Распаковывает данные из файла |
| `create_students` | Создает тестовые данные |
| `print_students` | Выводит данные студентов |
| `free_students` | Освобождает память |
| `filename` | `const char*` | Имя файла |
| `file` | `FILE*` | Файловый указатель |
| `count` | `int` | Количество структур |
| `i` | `int` | Индекс |



# код

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    int id;
    char name[50];
    float grade;
} Student;

int pack_data(const char* filename, Student* students, int count) {
    FILE* file = fopen(filename, "wb");
    if (!file) return 0;
    
    fwrite(&count, sizeof(int), 1, file);
    fwrite(students, sizeof(Student), count, file);
    
    fclose(file);
    return 1;
}

Student* unpack_data(const char* filename, int* count) {
    FILE* file = fopen(filename, "rb");
    if (!file) return NULL;
    
    fread(count, sizeof(int), 1, file);
    
    Student* students = (Student*)malloc(*count * sizeof(Student));
    fread(students, sizeof(Student), *count, file);
    
    fclose(file);
    return students;
}

Student* create_students(int count) {
    Student* students = (Student*)malloc(count * sizeof(Student));
    
    for (int i = 0; i < count; i++) {
        students[i].id = 1000 + i;
        snprintf(students[i].name, 50, "Student%d", i + 1);
        students[i].grade = 3.0 + (i * 0.5);
    }
    
    return students;
}

void print_students(Student* students, int count) {
    printf("Students data:\n");
    printf("ID   | Name           | Grade\n");
    printf("-----|----------------|------\n");
    
    for (int i = 0; i < count; i++) {
        printf("%4d | %-14s | %5.1f\n", 
               students[i].id, students[i].name, students[i].grade);
    }
}

void free_students(Student* students) {
    free(students);
}

int main() {
    const char* filename = "students.bin";
    int count = 5;
    
    printf("=== Data Packing/Unpacking ===\n\n");
    
    Student* original = create_students(count);
    printf("Original data:\n");
    print_students(original, count);
    
    if (pack_data(filename, original, count)) {
        printf("\nData packed successfully to %s\n", filename);
    } else {
        printf("\nError packing data\n");
        free_students(original);
        return 1;
    }
    
    int unpacked_count;
    Student* unpacked = unpack_data(filename, &unpacked_count);
    
    if (unpacked) {
        printf("\nUnpacked data:\n");
        print_students(unpacked, unpacked_count);
        
        printf("\nData integrity check: %s\n", 
               memcmp(original, unpacked, count * sizeof(Student)) == 0 ? "PASS" : "FAIL");
        
        free_students(unpacked);
    } else {
        printf("\nError unpacking data\n");
    }
    
    free_students(original);
    return 0;
}
```

# результат

<img width="328" height="225" alt="image" src="https://github.com/user-attachments/assets/a2e9cc63-703d-436f-82e2-4380ce4d26a5" />

<img width="409" height="218" alt="image" src="https://github.com/user-attachments/assets/72657bc5-e6c2-4aaa-a318-78767b2a6f09" />



# Задача 10: Генерация изображений

## постановка задачи

<img width="747" height="127" alt="image" src="https://github.com/user-attachments/assets/e399f6cc-51c5-444e-a983-ed8ec5f07bb7" />


# математическая модель

PPM формат: P6 → бинарный RGB

Пиксель: (r, g, b) ∈ [0, 255]³


Градиент: 

r(x,y) = 255 * x / width

g(x,y) = 255 * y / height

b(x,y) = 128

Случайный шум:

r(x,y) = random(0,255)

g(x,y) = random(0,255)

b(x,y) = random(0,255)


# индетификаторы

| Идентификатор | Тип | Назначение |
|---------------|-----|------------|
| `Pixel` | `struct` | Структура пикселя |
| `r, g, b` | `unsigned char` | Цветовые компоненты |
| `Image` | `struct` | Изображение |
| `width, height` | `int` | Размеры |
| `pixels` | `Pixel*` | Массив пикселей |
| `create_image` | Создает изображение |
| `free_image` | Освобождает память |
| `generate_gradient` | Генерирует градиент |
| `generate_noise` | Генерирует шум |
| `save_ppm` | Сохраняет в PPM |
| `print_image_info` | Выводит информацию |
| `width, height` | `int` | Размеры изображения |
| `type` | `int` | Тип генерации |
| `x, y` | `int` | Координаты пикселя |

# код
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

typedef struct {
    unsigned char r, g, b;
} Pixel;

typedef struct {
    int width;
    int height;
    Pixel* pixels;
} Image;

Image* create_image(int width, int height) {
    Image* img = (Image*)malloc(sizeof(Image));
    img->width = width;
    img->height = height;
    img->pixels = (Pixel*)malloc(width * height * sizeof(Pixel));
    return img;
}

void free_image(Image* img) {
    if (img) {
        free(img->pixels);
        free(img);
    }
}

void generate_gradient(Image* img) {
    for (int y = 0; y < img->height; y++) {
        for (int x = 0; x < img->width; x++) {
            Pixel* p = &img->pixels[y * img->width + x];
            p->r = (255 * x) / img->width;
            p->g = (255 * y) / img->height;
            p->b = 128;
        }
    }
}

void generate_noise(Image* img) {
    for (int i = 0; i < img->width * img->height; i++) {
        img->pixels[i].r = rand() % 256;
        img->pixels[i].g = rand() % 256;
        img->pixels[i].b = rand() % 256;
    }
}

int save_ppm(const char* filename, Image* img) {
    FILE* file = fopen(filename, "wb");
    if (!file) return 0;
    
    fprintf(file, "P6\n%d %d\n255\n", img->width, img->height);
    fwrite(img->pixels, sizeof(Pixel), img->width * img->height, file);
    
    fclose(file);
    return 1;
}

void print_image_info(Image* img, const char* type) {
    printf("Image: %dx%d, Type: %s\n", img->width, img->height, type);
    printf("First pixel: RGB(%d, %d, %d)\n", 
           img->pixels[0].r, img->pixels[0].g, img->pixels[0].b);
    printf("Last pixel: RGB(%d, %d, %d)\n", 
           img->pixels[img->width * img->height - 1].r,
           img->pixels[img->width * img->height - 1].g,
           img->pixels[img->width * img->height - 1].b);
}

int main() {
    srand(time(NULL));
    
    printf("=== Image Generation ===\n\n");
    
    int width = 256;
    int height = 256;
    
    // Градиентное изображение
    Image* gradient = create_image(width, height);
    generate_gradient(gradient);
    
    if (save_ppm("gradient.ppm", gradient)) {
        printf("Saved: gradient.ppm\n");
        print_image_info(gradient, "Gradient");
    }
    
    printf("\n");
    
    // Шумовое изображение
    Image* noise = create_image(width, height);
    generate_noise(noise);
    
    if (save_ppm("noise.ppm", noise)) {
        printf("Saved: noise.ppm\n");
        print_image_info(noise, "Random Noise");
    }
    
    free_image(gradient);
    free_image(noise);
    
    printf("\nFiles can be opened with image viewers supporting PPM format.\n");
    
    return 0;
}
```

# результат

<img width="375" height="232" alt="image" src="https://github.com/user-attachments/assets/b08a285e-a0fc-4ec8-8cd5-8d427330e972" />






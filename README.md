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

|---|---|---|
| Matrix | struct | Структура для представления матрицы |
| pdData | double** | Указатель на данные матрицы |
| iRows | int | Количество строк в матрице |
| iCols | int | Количество столбцов в матрице |
| pMat | Matrix* | Указатель на структуру матрицы |
| i | int | Счетчик строк |
| j | int | Счетчик столбцов |
| k | int | Внутренний счетчик для умножения |
| pResult | Matrix* | Указатель на результирующую матрицу |
| pMat1 | Matrix* | Указатель на первую матрицу |
| pMat2 | Matrix* | Указатель на вторую матрицу |
| dMat | double[][] | Двумерный массив для VLA |
| dResult | double[][] | Результирующий массив для VLA |
| dMat1 | double[][] | Первый массив для VLA операций |
| dMat2 | double[][] | Второй массив для VLA операций |
| iRows1 | int | Количество строк первой матрицы VLA |
| iCols1 | int | Количество столбцов первой матрицы VLA |
| iCols2 | int | Количество столбцов второй матрицы VLA |
| gsl_mat1 | gsl_matrix* | Указатель на GSL матрицу 1 |
| gsl_mat2 | gsl_matrix* | Указатель на GSL матрицу 2 |
| gsl_result | gsl_matrix* | Указатель на GSL результирующую матрицу |

# код

```c
#include <stdio.h>
#include <stdlib.h>
#include <gsl/gsl_matrix.h>

typedef struct {
    double **data;
    int rows;
    int cols;
} Matrix;

Matrix* create_matrix(int rows, int cols) {
    Matrix *mat = (Matrix*)malloc(sizeof(Matrix));
    mat->rows = rows;
    mat->cols = cols;
    mat->data = (double**)malloc(rows * sizeof(double*));
    for (int i = 0; i < rows; i++) {
        mat->data[i] = (double*)malloc(cols * sizeof(double));
    }
    return mat;
}

void free_matrix(Matrix *mat) {
    for (int i = 0; i < mat->rows; i++) {
        free(mat->data[i]);
    }
    free(mat->data);
    free(mat);
}

void fill_matrix(Matrix *mat) {
    for (int i = 0; i < mat->rows; i++) {
        for (int j = 0; j < mat->cols; j++) {
            mat->data[i][j] = i * mat->cols + j + 1;
        }
    }
}

void print_matrix(const Matrix *mat) {
    for (int i = 0; i < mat->rows; i++) {
        for (int j = 0; j < mat->cols; j++) {
            printf("%6.1f ", mat->data[i][j]);
        }
        printf("\n");
    }
}

Matrix* transpose_dynamic(const Matrix *mat) {
    Matrix *result = create_matrix(mat->cols, mat->rows);
    for (int i = 0; i < mat->rows; i++) {
        for (int j = 0; j < mat->cols; j++) {
            result->data[j][i] = mat->data[i][j];
        }
    }
    return result;
}

Matrix* add_matrices_dynamic(const Matrix *mat1, const Matrix *mat2) {
    if (mat1->rows != mat2->rows || mat1->cols != mat2->cols) {
        printf("Error: matrix dimensions don't match!\n");
        return NULL;
    }
    Matrix *result = create_matrix(mat1->rows, mat1->cols);
    for (int i = 0; i < mat1->rows; i++) {
        for (int j = 0; j < mat1->cols; j++) {
            result->data[i][j] = mat1->data[i][j] + mat2->data[i][j];
        }
    }
    return result;
}

Matrix* multiply_matrices_dynamic(const Matrix *mat1, const Matrix *mat2) {
    if (mat1->cols != mat2->rows) {
        printf("Error: incompatible dimensions for multiplication!\n");
        return NULL;
    }
    Matrix *result = create_matrix(mat1->rows, mat2->cols);
    for (int i = 0; i < mat1->rows; i++) {
        for (int j = 0; j < mat2->cols; j++) {
            result->data[i][j] = 0;
            for (int k = 0; k < mat1->cols; k++) {
                result->data[i][j] += mat1->data[i][k] * mat2->data[k][j];
            }
        }
    }
    return result;
}

void transpose_vla(int rows, int cols, double mat[rows][cols], double result[cols][rows]) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            result[j][i] = mat[i][j];
        }
    }
}

void add_matrices_vla(int rows, int cols, 
                     double mat1[rows][cols], 
                     double mat2[rows][cols], 
                     double result[rows][cols]) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            result[i][j] = mat1[i][j] + mat2[i][j];
        }
    }
}

void multiply_matrices_vla(int rows1, int cols1, int cols2,
                          double mat1[rows1][cols1],
                          double mat2[cols1][cols2],
                          double result[rows1][cols2]) {
    for (int i = 0; i < rows1; i++) {
        for (int j = 0; j < cols2; j++) {
            result[i][j] = 0;
            for (int k = 0; k < cols1; k++) {
                result[i][j] += mat1[i][k] * mat2[k][j];
            }
        }
    }
}

void print_vla_matrix(int rows, int cols, double mat[rows][cols]) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            printf("%6.1f ", mat[i][j]);
        }
        printf("\n");
    }
}

void verify_with_gsl() {
    printf("=== GSL VERIFICATION ===\n");
    
    gsl_matrix *gsl_mat1 = gsl_matrix_alloc(2, 3);
    gsl_matrix *gsl_mat2 = gsl_matrix_alloc(3, 2);
    
    for (int i = 0; i < 2; i++) {
        for (int j = 0; j < 3; j++) {
            gsl_matrix_set(gsl_mat1, i, j, i * 3 + j + 1);
        }
    }
    
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 2; j++) {
            gsl_matrix_set(gsl_mat2, i, j, i * 2 + j + 1);
        }
    }
    
    gsl_matrix *gsl_result = gsl_matrix_alloc(2, 2);
    gsl_blas_dgemm(CblasNoTrans, CblasNoTrans, 1.0, gsl_mat1, gsl_mat2, 0.0, gsl_result);
    
    printf("GSL multiplication result:\n");
    for (int i = 0; i < 2; i++) {
        for (int j = 0; j < 2; j++) {
            printf("%6.1f ", gsl_matrix_get(gsl_result, i, j));
        }
        printf("\n");
    }
    
    gsl_matrix_free(gsl_mat1);
    gsl_matrix_free(gsl_mat2);
    gsl_matrix_free(gsl_result);
}

int main() {
    printf("=== DYNAMIC MATRICES ===\n");
    
    Matrix *mat1 = create_matrix(2, 3);
    Matrix *mat2 = create_matrix(2, 3);
    Matrix *mat3 = create_matrix(3, 2);
    
    fill_matrix(mat1);
    fill_matrix(mat2);
    fill_matrix(mat3);
    
    printf("Matrix 1:\n");
    print_matrix(mat1);
    printf("Matrix 2:\n");
    print_matrix(mat2);
    printf("Matrix 3:\n");
    print_matrix(mat3);
    
    Matrix *transposed = transpose_dynamic(mat1);
    printf("Transposed matrix 1:\n");
    print_matrix(transposed);
    
    Matrix *sum = add_matrices_dynamic(mat1, mat2);
    printf("Matrix 1 + Matrix 2:\n");
    print_matrix(sum);
    
    Matrix *product = multiply_matrices_dynamic(mat1, mat3);
    printf("Matrix 1 * Matrix 3:\n");
    print_matrix(product);
    
    printf("\n=== VLA MATRICES ===\n");
    
    double vla1[2][3] = {{1, 2, 3}, {4, 5, 6}};
    double vla2[2][3] = {{6, 5, 4}, {3, 2, 1}};
    double vla3[3][2] = {{1, 2}, {3, 4}, {5, 6}};
    
    double vla_transposed[3][2];
    double vla_sum[2][3];
    double vla_product[2][2];
    
    transpose_vla(2, 3, vla1, vla_transposed);
    printf("VLA transposed:\n");
    print_vla_matrix(3, 2, vla_transposed);
    
    add_matrices_vla(2, 3, vla1, vla2, vla_sum);
    printf("VLA sum:\n");
    print_vla_matrix(2, 3, vla_sum);
    
    multiply_matrices_vla(2, 3, 2, vla1, vla3, vla_product);
    printf("VLA product:\n");
    print_vla_matrix(2, 2, vla_product);
    
    verify_with_gsl();
    
    free_matrix(mat1);
    free_matrix(mat2);
    free_matrix(mat3);
    free_matrix(transposed);
    free_matrix(sum);
    free_matrix(product);
    
    return 0;
}
```
# результат 
<img width="560" height="118" alt="image" src="https://github.com/user-attachments/assets/d0c4d8c4-cba0-4879-be31-a5e234876d17" />

# Задача 2: Упрощённый парсер JSON


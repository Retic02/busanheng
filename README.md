//20211068 최정훈
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define LEN_MIN  15   
#define LEN_MAX  50
#define STM_MIN  0    
#define STM_MAX  5
#define PROB_MIN  10  
#define PROB_MAX  90
#define AGGRO_MIN  0  
#define AGGRO_MAX  5
#define MOVE_LEFT  1
#define MOVE_STAY  0
#define ATK_NONE  0
#define ATK_CITIZEN  1
#define ATK_DONGSEOK  2
#define ACTION_REST  0
#define ACTION_PROVOKE  1
#define ACTION_PULL  2

void print_banner() {
    printf("---------------------\n");
    printf("|      부산헹       |\n");
    printf("---------------------\n");
    printf("   ()     ()     ()   \n");
}

int get_madongseok_stamina() {
    int madongseok_stamina;
    while (1) {
        printf("madongseok stamina(%d~%d)>>", STM_MIN, STM_MAX);
        if (scanf_s("%d", &madongseok_stamina) == 1 && madongseok_stamina >= STM_MIN && madongseok_stamina <= STM_MAX) {
            break;
        }
    }
    return madongseok_stamina;
}

int get_train_length() {
    int train_length;
    while (1) {
        printf("train length(%d~%d) >> ", LEN_MIN, LEN_MAX);
        if (scanf_s("%d", &train_length) == 1 && train_length >= LEN_MIN && train_length <= LEN_MAX) {
            break;
        }
    }
    return train_length;
}

int get_probability() {
    int probability;
    while (1) {
        printf("percentile probability 'p'(%d~%d) >> ", PROB_MIN, PROB_MAX);
        if (scanf_s("%d", &probability) == 1 && probability >= PROB_MIN && probability <= PROB_MAX) {
            break;
        }
    }
    return probability;
}

void print_train(int train_length) {
    for (int i = 0; i < train_length; i++) {
        printf("#");
    }
    printf("\n");
}

void print_train_with_chars(int train_length, int cpos, int zpos, int mpos) {
    printf("#");
    for (int i = 1; i <= train_length - 2; i++) {
        if (i == cpos) {
            printf("C");
        }
        else if (i == zpos) {
            printf("Z");
        }
        else if (i == mpos) {
            printf("M");
        }
        else {
            printf(" ");
        }
    }
    printf("#\n");
}

void print_move_status(int train_length, int c_move, int* cpos, int* zpos, int* mpos, int* aggro_c, int* aggro_m, int iter, int probability, int* zombie_pulled, int* cprev, int* mprev, int* zprev, int* aggro_c_prev, int* aggro_m_prev) {
    if (c_move < (100 - probability) && *cpos > 1) {
        if (*aggro_c_prev == AGGRO_MAX && *aggro_c == AGGRO_MAX) {
            printf("citizen : %d -> %d (aggro : %d)\n", *cprev, *cpos, AGGRO_MAX);
        }
        else {
            printf("citizen : %d -> %d (aggro : %d -> %d)\n", *cprev, *cpos, *aggro_c_prev, *aggro_c);
        }
    }
    else {
        if (*aggro_c_prev == AGGRO_MIN && *aggro_c == AGGRO_MIN) {
            printf("citizen : stay %d (aggro : %d)\n", *cpos, AGGRO_MIN);
        }
        else {
            printf("citizen : stay %d (aggro : %d -> %d)\n", *cpos, *aggro_c_prev, *aggro_c);
        }
    }

    if (*zombie_pulled) {
        printf("zombie : stay %d (pulled)\n", *zpos);
    }
    else if (iter % 2 != 0) {
        if (*zprev == *zpos) {
            printf("zombie : stay %d\n", *zpos);
        }
        else if (*zprev == *zpos + 1) {
            printf("zombie : %d -> %d\n", *zprev, *zpos);
        }
        else if (*zprev == *zpos - 1) {
            printf("zombie : %d -> %d\n", *zprev, *zpos);
        }
    }
    else {
        printf("zombie : stay %d (cannot move)\n", *zpos);
    }
    *zombie_pulled = 0;
}

void move_phase(int train_length, int* cpos, int* zpos, int* mpos, int* aggro_c, int* aggro_m, int iter, int probability, int* zombie_pulled, int* cprev, int* mprev, int* zprev, int* aggro_c_prev, int* aggro_m_prev, int c_move, int* stamina) {
   
    *cprev = *cpos;
    *mprev = *mpos;
    *zprev = *zpos;
    *aggro_c_prev = *aggro_c;
    *aggro_m_prev = *aggro_m;
    if (c_move < (100 - probability) && *cpos > 1) {
        (*cpos)--;
        (*aggro_c)++;
        if (*aggro_c > AGGRO_MAX) {
            (*aggro_c) = AGGRO_MAX;
        }
        else {
        }
    }
    else {
        (*aggro_c)--;
        if (*aggro_c < AGGRO_MIN) {
            (*aggro_c) = AGGRO_MIN;
        }
    }

    if (*zombie_pulled) {
        *zpos = *zpos;
    }
    else if (iter % 2 != 0) {
        if (*cpos != *zpos - 1 && *aggro_c >= *aggro_m) {
            (*zpos)--;
        }
        else if (*mpos != *zpos + 1 && *aggro_c < *aggro_m) {
            (*zpos)++;
        }
        else if (*cpos == *zpos - 1 && *aggro_c >= *aggro_m) {
            *zpos = *zpos;
        }
        else if (*mpos == *zpos + 1 && *aggro_c < *aggro_m) {
            *zpos = *zpos;
        }
        else if (*cpos == *zpos - 1 && *mpos == *zpos + 1) {
            *zpos = *zpos;
        }
    }
    print_train(train_length);
    print_train_with_chars(train_length, *cpos, *zpos, *mpos);
    print_train(train_length);

    print_move_status(train_length, c_move, cpos, zpos, mpos, aggro_c, aggro_m, iter, probability, zombie_pulled, cprev, mprev, zprev, aggro_c_prev, aggro_m_prev);

    if (*mpos == *zpos + 1) {
        printf("madongseok move (0: stay) >> ");
        while (1) {
            int m_move;
            if (scanf_s("%d", &m_move) == 1 && m_move == MOVE_STAY) {
                break;
            }
        }
        (*aggro_m)--;
        if (*aggro_m < AGGRO_MIN) (*aggro_m) = AGGRO_MIN;
    }
    else {
        int m_move;
        printf("madongseok move (0: stay, 1: left) >> ");
        while (1) {
            if (scanf_s("%d", &m_move) == 1 && (m_move == MOVE_STAY || m_move == MOVE_LEFT)) {
                break;
            }
        }
        if (m_move == MOVE_LEFT && *mpos > 1) {
            (*mpos)--;
            (*aggro_m)++;
            if (*aggro_m > AGGRO_MAX) (*aggro_m) = AGGRO_MAX;
        }
        else {
            *mpos = *mpos;
            (*aggro_m)--;
            if (*aggro_m < AGGRO_MIN) (*aggro_m) = AGGRO_MIN;
        }
    }
    print_train(train_length);
    print_train_with_chars(train_length, *cpos, *zpos, *mpos);
    print_train(train_length);

    if (*mprev == *mpos) {
        printf("madongseok : stay %d (aggro : %d -> %d)\n", *mpos, *aggro_m_prev, *aggro_m);
    }
    else {
        printf("madongseok : %d -> %d (aggro : %d -> %d, stamina : %d)\n", *mprev, *mpos, *aggro_m_prev, *aggro_m, *stamina);
    }
}

void print_action_status(int* cpos, int* stamina,int* aggro_c, int* aggro_m, int* zpos, int* mpos) {
    if (*cpos == 1) {
        printf("YOU WIN! Citizen(s) escaped to the next train\n");
        exit(0);
    }
    else {
        printf("citizen does nothing.\n");
    }

    if (*zpos == *cpos + 1 && *zpos == *mpos + 1) { 
        if (*aggro_c > *aggro_m) {
            printf("zombie attacks citizen!\n");
            printf("GAME OVER! Citizen dead...\n");
            exit(0);
        }
        else {
            printf("zombie attacks madongseok (aggro : %d vs. %d, madongseok stamina %d -> %d)\n", *aggro_c, *aggro_m, *stamina + 1, *stamina);
            if (*stamina <= STM_MIN) {
                printf("GAME OVER! Madongseok dead...\n");
                exit(0);
            }
        }
    }
    else if (*zpos == *cpos + 1) { 
        printf("zombie attacks citizen!\n");
        printf("GAME OVER! Citizen dead...\n");
        exit(0);
    }
    else if (*zpos == *mpos - 1) {
        printf("zombie attacks madongseok (aggro : %d vs. %d, madongseok stamina %d -> %d)\n", *aggro_c, *aggro_m, *stamina + 1, *stamina);
        if (*stamina <= STM_MIN) {
            printf("GAME OVER! Madongseok dead...\n");
            exit(0);
        }
        else {
            printf("zombie does nothing.\n");
        }
    }
}



void action_phase(int train_length, int* cpos, int* zpos, int* mpos, int* aggro_c, int* aggro_m, int* stamina, int probability, int* zombie_pulled, int* aggro_m_prev, int* sta_prev) {

    if (*zpos == *cpos + 1 && *zpos == *mpos + 1) { 
        if (*aggro_c > *aggro_m) {
        }
        else {
            (*stamina)--;
        }
    }
    else if (*zpos == *cpos + 1) { 
    }
    else if (*zpos == *mpos - 1) { 
        (*stamina)--;
    }

    print_action_status(cpos, stamina, aggro_c, aggro_m, zpos, mpos);

    int action;
    *aggro_m_prev = *aggro_m;
    *sta_prev = *stamina;
    if (*mpos == *zpos + 1) { 
        printf("madongseok action (0: rest, 1: provoke, 2: pull) >> ");
        while (1) {
            if (scanf_s("%d", &action) == 1 && (action == ACTION_REST || action == ACTION_PROVOKE || action == ACTION_PULL)) {
                break;
            }
        }
        if (action == ACTION_REST) {
            (*aggro_m)--;
            if (*aggro_m < 1) *aggro_m = 1;
            (*stamina)++;
            printf("madongseok rests... \n");
            printf("madongseok : %d (aggro: %d -> %d, stamina: %d -> %d)\n", *mpos, *aggro_m_prev, *aggro_m, *stamina - 1, *stamina);
        }
        else if (action == ACTION_PROVOKE) {
            *aggro_m = AGGRO_MAX;
            printf("madongseok provokes zombie... Aggro: %d\n", *aggro_m);
            printf("madongseok : %d (aggro : %d -> %d, stamina : %d)\n", *mpos, *aggro_m_prev, *aggro_m, *stamina);
        }
        else if (action == ACTION_PULL) {
            (*aggro_m) += 2;
            (*stamina)--;
            int pull_success = rand() % 100;
            if (pull_success < (100 - probability)) {
                *zombie_pulled = 1;
                printf("madongseok pulled zombie.... Next turn, it can't move.\n");
                printf("madongseok : %d (aggro : %d -> %d, stamina: %d -> %d)\n", *mpos, *aggro_m_prev, *aggro_m, *sta_prev, *stamina);
            }
            else {
                printf("madongseok fails to pull zombie.\n");
            }
            if (*stamina <= STM_MIN) {
                printf("GAME OVER! madongseok dead...\n");
                exit(0);
            }
        }
    }
    else { 
        printf("Madongseok action (0: rest, 1: provoke) >> ");
        while (1) {
            if (scanf_s("%d", &action) == 1 && (action == ACTION_REST || action == ACTION_PROVOKE)) {
                break;
            }
        }
        if (action == ACTION_REST) {
            (*aggro_m)--;
            if (*aggro_m < 1) *aggro_m = 1;
            (*stamina)++;
            printf("Madongseok rests. Aggro: %d, Stamina: %d\n", *aggro_m, *stamina);
        }
        else if (action == ACTION_PROVOKE) {
            *aggro_m = AGGRO_MAX;
            printf("Madongseok provokes. Aggro: %d\n", *aggro_m);
        }
    }
}

void game_loop(int train_length, int madongseok_stamina, int probability) {
    int iter = 1;
    int cpos = train_length - 6;
    int zpos = train_length - 3;
    int mpos = train_length - 2;
    int cprev, zprev, mprev;
    int aggro_c_prev, aggro_m_prev;
    int aggro_c = 1, aggro_m = 1;
    int stamina = madongseok_stamina;
    int zombie_pulled = 0;
    int c_move;
    int sta_prev;

    while (1) {
        cprev = cpos;
        zprev = zpos;
        mprev = mpos;
        aggro_c_prev = aggro_c;
        aggro_m_prev = aggro_m;
        c_move = rand() % 100;

        move_phase(train_length, &cpos, &zpos, &mpos, &aggro_c, &aggro_m, iter, probability, &zombie_pulled, &cprev, &mprev, &zprev, &aggro_c_prev, &aggro_m_prev, c_move, &stamina);

        action_phase(train_length, &cpos, &zpos, &mpos, &aggro_c, &aggro_m, &stamina, probability, &zombie_pulled, &aggro_m_prev, &sta_prev);

        iter++;
    }
}

int main(void) {
    srand(time(NULL));

    print_banner();

    printf("\n");
    
    int train_length = get_train_length();
    int madongseok_stamina = get_madongseok_stamina();
    int probability = get_probability();
    
    print_train(train_length);
    print_train_with_chars(train_length, train_length - 6, train_length - 3, train_length - 2);
    print_train(train_length);

    printf("\n");

    game_loop(train_length, madongseok_stamina, probability);

    print_banner();
    return 0;
}

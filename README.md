#include <conio.h>
#include <string.h>
#include <time.h>

#define MAX 50

struct bank
{
    int accNo;
    char name[50];
    char password[20];
    float balance;
    float loan;
};

struct transaction
{
    int accNo;
    char type[30];
    float amount;
    char time[40];
};

struct bank b[MAX];
int total = 0;

/* ---------- TIME ---------- */
void getTime(char *t)
{
    time_t now = time(NULL);
    char *tmp = ctime(&now);
    tmp[strlen(tmp) - 1] = '\0';   // remove newline
    strcpy(t, tmp);
}

/* ---------- FILE LOAD ---------- */
void loadData()
{
    FILE *fp = fopen("bank.dat", "rb");
    if(fp)
    {
        fread(&total, sizeof(int), 1, fp);
        fread(b, sizeof(struct bank), total, fp);
        fclose(fp);
    }
}

/* ---------- FILE SAVE ---------- */
void saveData()
{
    FILE *fp = fopen("bank.dat", "wb");
    if(fp == NULL)
    {
        printf("File Error!\n");
        return;
    }
    fwrite(&total, sizeof(int), 1, fp);
    fwrite(b, sizeof(struct bank), total, fp);
    fclose(fp);
}

/* ---------- TRANSACTION SAVE ---------- */
void saveTransaction(int acc, char type[], float amt)
{
    struct transaction t;
    FILE *fp = fopen("transaction.dat", "ab");
    if(fp == NULL) return;

    t.accNo = acc;
    strcpy(t.type, type);
    t.amount = amt;
    getTime(t.time);

    fwrite(&t, sizeof(t), 1, fp);
    fclose(fp);
}

/* ---------- CREATE ACCOUNT ---------- */
void createAccount()
{
    if(total >= MAX)
    {
        printf("Account limit reached!\n");
        return;
    }

    b[total].accNo = total + 1001;

    printf("Enter Name: ");
    scanf(" %[^\n]", b[total].name);

    printf("Set Password: ");
    scanf("%s", b[total].password);

    b[total].balance = 0;
    b[total].loan = 0;

    total++;
    saveData();

    printf("Account Created!\nAccount No: %d\n", b[total - 1].accNo);
}

/* ---------- LOGIN ---------- */
int login()
{
    int acc;
    char pass[20];

    printf("Account No: ");
    scanf("%d", &acc);
    printf("Password: ");
    scanf("%s", pass);

    for(int i = 0; i < total; i++)
        if(b[i].accNo == acc && strcmp(b[i].password, pass) == 0)
            return i;

    return -1;
}

/* ---------- FEATURES ---------- */
void balanceCheck(int i)
{
    printf("Balance: %.2f\nLoan: %.2f\n", b[i].balance, b[i].loan);
}

void deposit(int i)
{
    float amt;
    printf("Amount: ");
    scanf("%f", &amt);

    if(amt <= 0) return;

    b[i].balance += amt;
    saveTransaction(b[i].accNo, "Deposit", amt);
    saveData();
    printf("Deposit Successful\n");
}

void withdraw(int i)
{
    float amt;
    printf("Amount: ");
    scanf("%f", &amt);

    if(amt <= 0 || amt > b[i].balance)
    {
        printf("Invalid Amount\n");
        return;
    }

    b[i].balance -= amt;
    saveTransaction(b[i].accNo, "Withdraw", amt);
    saveData();
    printf("Withdraw Successful\n");
}

void transfer(int i)
{
    int acc;
    float amt;

    printf("Receiver Acc No: ");
    scanf("%d", &acc);
    printf("Amount: ");
    scanf("%f", &amt);

    if(amt <= 0 || amt > b[i].balance) return;

    for(int j = 0; j < total; j++)
    {
        if(b[j].accNo == acc && j != i)
        {
            b[i].balance -= amt;
            b[j].balance += amt;
            saveTransaction(b[i].accNo, "Transfer", amt);
            saveData();
            printf("Transfer Successful\n");
            return;
        }
    }
    printf("Account Not Found\n");
}

void loanTake(int i)
{
    float amt;
    printf("Loan Amount: ");
    scanf("%f", &amt);

    if(amt <= 0 || b[i].loan + amt > 50000)
    {
        printf("Loan Limit Exceeded\n");
        return;
    }

    b[i].loan += amt;
    b[i].balance += amt;
    saveTransaction(b[i].accNo, "Loan", amt);
    saveData();
    printf("Loan Approved\n");
}

void repayLoan(int i)
{
    float amt;
    printf("Repay Amount: ");
    scanf("%f", &amt);

    if(amt <= 0 || amt > b[i].balance || amt > b[i].loan)
    {
        printf("Invalid Amount\n");
        return;
    }

    b[i].balance -= amt;
    b[i].loan -= amt;
    saveTransaction(b[i].accNo, "Loan Repay", amt);
    saveData();
    printf("Loan Repaid\n");
}

void myTransactions(int acc)
{
    struct transaction t;
    FILE *fp = fopen("transaction.dat", "rb");

    if(fp == NULL)
    {
        printf("No Transactions Found\n");
        return;
    }

    while(fread(&t, sizeof(t), 1, fp))
        if(t.accNo == acc)
            printf("%s | %.2f | %s\n", t.type, t.amount, t.time);

    fclose(fp);
}

/* ---------- ADMIN ---------- */
void adminPanel()
{
    char u[20], p[20];
    printf("Admin Username: ");
    scanf("%s", u);
    printf("Admin Password: ");
    scanf("%s", p);

    if(strcmp(u, "admin") || strcmp(p, "admin123"))
    {
        printf("Invalid Admin Login\n");
        return;
    }

    int ch;
    while(1)
    {
        printf("\n1.All Accounts\n2.Transaction History\n3.Exit\n");
        scanf("%d", &ch);

        if(ch == 1)
            for(int i = 0; i < total; i++)
                printf("%d | %s | %.2f\n", b[i].accNo, b[i].name, b[i].balance);

        else if(ch == 2)
        {
            struct transaction t;
            FILE *fp = fopen("transaction.dat", "rb");
            if(fp == NULL) return;

            while(fread(&t, sizeof(t), 1, fp))
                printf("%d | %s | %.2f | %s\n",
                       t.accNo, t.type, t.amount, t.time);
            fclose(fp);
        }
        else break;
    }
}

/* ---------- MAIN ---------- */
int main()
{
    int ch, u;
    loadData();

    while(1)
    {
        printf("\n1.Create Account\n2.Login\n3.Admin Panel\n4.Exit\n");
        scanf("%d", &ch);

        if(ch == 1) createAccount();

        else if(ch == 2)
        {
            u = login();
            if(u == -1) continue;

            int c;
            while(1)
            {
                printf("\n1.Balance\n2.Deposit\n3.Withdraw\n4.Transfer\n5.Loan\n6.Repay Loan\n7.Transactions\n8.Logout\n");
                scanf("%d", &c);

                if(c == 1) balanceCheck(u);
                else if(c == 2) deposit(u);
                else if(c == 3) withdraw(u);
                else if(c == 4) transfer(u);
                else if(c == 5) loanTake(u);
                else if(c == 6) repayLoan(u);
                else if(c == 7) myTransactions(b[u].accNo);
                else break;
            }
        }
        else if(ch == 3) adminPanel();
        else break;
    }
    getch();
}

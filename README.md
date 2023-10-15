# C_project_01

## Study planner:
Write program to print your daily schedule. The program reads your schedule from a CSV file.
The header row has four headings: Day, Time, Course, Room. Data rows need to appear in any specific order.
Program asks user to choose a day and prints the day’s schedule starting with earliest time in ascending order.
If there are no classes for the day program prints “No classes today”. If user enters “stop” as a day the program stops.
```
Day, Time, Course, Room
Monday, 9:00, C programming, KNE555
Tuesday, 13:00, Network, KMD666
Wednesday, 10:00, Academic writing, KMD777
Monday, 13:00, Math, KME663
Friday, 14:00, Routing, KMD666
Tuesday, 9:00, Java, KNE556
Friday, 8:00, Online course, N/A

For example:
Enter day: Tuesday
On Tuesday you have:
9:00 Java, KNE556
13:00 Network, KMD666
Enter day: Thursday
No classes today
```
## Code
```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <ctype.h>
#include <stdbool.h>

#define FILE_NAME "schedule.csv"
#define LINE_SIZE 200
#define MAX_DAILY_LESSONS 10
#define DAY_LENGTH 10
#define TIME_LENGTH 10
#define COURSE_LENGTH 100
#define ROOM_LENGTH 10

// Define the structure to stored in CSV file
typedef struct node
{
    char day[DAY_LENGTH];
    char time[TIME_LENGTH];
    char course[COURSE_LENGTH];
    char room[ROOM_LENGTH];
} schedule;

schedule *allocateMemory(char *line);
int sortByTime(const void *a, const void *b);

int main()
{
    FILE *file;
    int totalLesson = 0;
    const char dayOfWeek[7][10] = {"Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"};
    // read csv file
    file = fopen(FILE_NAME, "r");
    if (file != NULL)
    {
        // count how many valid data lines in order to declare enough space for weeklySchedule
        int countValidLines = 0;
        char c;
        while (!feof(file))
        {
            char line[LINE_SIZE];
            if (fgets(line, sizeof(line), file))
            {
                if (strchr(line, ':') != NULL)
                {
                    countValidLines++;
                }
            }
        }
        // printf("Total valid data lines: %d\n", countValidLines);
        // initialize weeklySchedule that holds the total number of weekly lessons based on countValidLines
        schedule *weeklySchedule[countValidLines];
        rewind(file); // Set position of stream to the beginning
        while (!feof(file))
        {
            char line[LINE_SIZE];
            if (fgets(line, sizeof(line), file))
            {
                // printf(">>>> Here\n");
                schedule *newNode = allocateMemory(line);
                if (newNode != NULL) // data was OK and a new node has been created
                {
                    // printf("%p\n", newNode);
                    weeklySchedule[totalLesson] = newNode;
                    totalLesson++;
                }
            }
        }
        // printf("Total lessons: %d\n", totalLesson);
        //  close the file
        fclose(file);

        // Print the data list for testing purpose
        for (int i = 0; i < totalLesson; i++)
        {
            printf("%s, %s, %s, %s", weeklySchedule[i]->day, weeklySchedule[i]->time, weeklySchedule[i]->course, weeklySchedule[i]->room);
        }
        // keep asking user unless "stop" entered
        char weekday[11];
        printf("Enter day or 'stop' to quit: ");
        while (fgets(weekday, sizeof(weekday), stdin) == NULL)
        {
            fprintf(stderr, "Error reading input\n");
            printf("Enter day or 'stop' to quit: ");
        }
        // control overflow safe for fgets
        if (strchr(weekday, '\n') == NULL)
        {
            // empty input buffer
            while (getchar() != '\n')
                ;
        }
        // replace \n from weekday with \0
        if (weekday[strlen(weekday) - 1] == '\n')
        {
            weekday[strlen(weekday) - 1] = '\0';
        }

        while (strcmp(weekday, "stop") != 0)
        {
            // convert the input word to lower case
            int i = 0;
            while (weekday[i] != '\0')
            {
                weekday[i] = tolower(weekday[i]);
                i++;
            }
            // convert back the 1st letter of weekday to upper case
            weekday[0] = toupper(weekday[0]);
            // printf("%s\n", weekday);
            bool mistyped = true;
            for (int i = 0; i < 7; i++)
            {
                // check if input day is mistyped or not
                if (strcmp(dayOfWeek[i], weekday) == 0)
                {
                    mistyped = false;
                    break;
                }
            }
            if (!mistyped) // lookup data if weekday is correct
            {
                // printf("day checked ok\n");
                bool scheduleFound = false;
                schedule todayClass[MAX_DAILY_LESSONS];
                int lessonNum = 0;
                for (int i = 0; i < totalLesson; i++)
                {
                    const schedule *tmp = weeklySchedule[i]; // convert non-const type to const type for strcmp and strcpy
                    if (strcmp(weekday, tmp->day) == 0)
                    {
                        if (lessonNum < MAX_DAILY_LESSONS) // make sure total lessons for a day is not exceed the defined total
                        {
                            scheduleFound = true;
                            strcpy(todayClass[lessonNum].day, tmp->day);
                            strcpy(todayClass[lessonNum].time, tmp->time);
                            strcpy(todayClass[lessonNum].course, tmp->course);
                            strcpy(todayClass[lessonNum].room, tmp->room);
                            lessonNum++;
                        }
                        else
                        {
                            printf("Exceed the total number of daily lessons.\n");
                        }
                    }
                }
                // printf("%d lessonNum\n", lessonNum);
                if (scheduleFound) // found classes for today
                {
                    // sort today's lessons by time
                    qsort(todayClass, lessonNum, sizeof(schedule), sortByTime);
                    // print out today's lessons
                    printf("\nOn %s you have: \n", todayClass[0].day);
                    for (int i = 0; i < lessonNum; i++)
                    {
                        printf("%5s %s, %s", todayClass[i].time, todayClass[i].course, todayClass[i].room);
                    }
                    printf("\n");
                }
                else // today's classes not found
                {
                    printf("No classes today\n\n");
                }
            }

            //
            printf("Enter day or 'stop' to quit: ");
            while (fgets(weekday, sizeof(weekday), stdin) == NULL)
            {
                fprintf(stderr, "Error reading input\n");
                printf("Enter day or 'stop' to quit: ");
            }
            // control overflow safe for fgets
            if (strchr(weekday, '\n') == NULL)
            {
                // empty input buffer
                while (getchar() != '\n')
                    ;
            }
            // replace \n from weekday with \0
            if (weekday[strlen(weekday) - 1] == '\n')
            {
                weekday[strlen(weekday) - 1] = '\0';
            }
        }
    }
    else
    {
        fprintf(stderr, "Unable to open file %s for reading\n", FILE_NAME);
        exit(1);
    }
    // free memory before exiting

    return 0;
}

schedule *allocateMemory(char *line)
{
    // check if data is in correct format
    if (strchr(line, ':') != NULL)
    {
        char *token;
        token = strtok(line, ",");
        if (token != NULL)
        {
            char daytmp[10];
            strncpy(daytmp, token, sizeof(daytmp));
            token = strtok(NULL, ",");
            if (token != NULL)
            {
                char timetmp[10];
                strncpy(timetmp, token, sizeof(timetmp));
                token = strtok(NULL, ",");
                if (token != NULL)
                {
                    char coursetmp[100];
                    strncpy(coursetmp, token, sizeof(coursetmp));
                    token = strtok(NULL, ",");
                    if (token != NULL)
                    {
                        char roomtmp[10];
                        strncpy(roomtmp, token, sizeof(roomtmp));
                        // move all tmp data to new node
                        schedule *newNode = (schedule *)malloc(sizeof(schedule));
                        if (newNode == NULL)
                        {
                            fprintf(stderr, "Memory allocation failed.\n");
                            exit(1);
                        }
                        // memory allocation succeeded
                        strncpy(newNode->day, daytmp, sizeof(newNode->day));
                        strncpy(newNode->time, timetmp, sizeof(newNode->time));
                        strncpy(newNode->course, coursetmp, sizeof(newNode->course));
                        strncpy(newNode->room, roomtmp, sizeof(newNode->room));
                        return newNode;
                    }
                }
            }
        }
    }
    return NULL;
}

int sortByTime(const void *a, const void *b)
{
    int cal = strcmp((*(schedule *)b).time, (*(schedule *)a).time);
    if (cal < 0)
        return -1;
    else if (cal > 0)
        return 1;
}
```

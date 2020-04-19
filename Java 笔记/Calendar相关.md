Java中有两个表示时间的类，一个是Date，还有一个是LocalDate，一下主要介绍LocalDate

---

## LocalDate语句

`LocalDate date = LocalDate.now()` 获得当前日期，需要通过一下两个方法获得当前月，日

- `int month = date.getMonthValue()`
  `int day = date.getDayOfMonth()`

`DayOfWeek weekday = data.minusDays(today - 1)`日期减去（today-1）天，回到当月第一天
`int value = weekday.getValue()`获得当前星期几，1=Monday，7=Sunday
`getMonthValue()`获得当前月份值
`plusDays(int n)`当前日期加n天



```java
import java.time.*;

public class CalendarTest {
    public static void main(String[] args) {
        LocalDate date = LocalDate.now();
        //get today's date
        int month = date.getMonthValue();
        int today = date.getDayOfMonth();
        //minus the day of month to start from the first day
        date = date.minusDays(today - 1);
        // get the value of the day in that week
        DayOfWeek weekday = date.getDayOfWeek();
        int value = weekday.getValue();
        System.out.println("Mon Tue Wed Thu Fri Sat Sun");
        for(int i = 1; i < value; i++)
            System.out.print("    ");
        while(date.getMonthValue() == month)
        {
            System.out.printf("%3d",date.getDayOfMonth());
            if(date.getDayOfMonth() == today)
                System.out.print("*");
            else
                System.out.print(" ");
            date = date.plusDays(1);
            if(date.getDayOfWeek().getValue() == 1)
                System.out.println();
        }
        if(date.getDayOfWeek().getValue() != 1)System.out.println();
    }
}
```


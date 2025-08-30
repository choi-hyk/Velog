[Velog로 가기](https://velog.io/@choi-hyk/GitHub-Pages-캘린더-구현하기)

released at 2025-07-22 15:32:29 KST

updated at 2025-08-28 03:16:52 KST

|[github pages](https://velog.io/tags/github-pages)||----|

## 📅캘린더 구현하기

저번에는 프로필 페이지에서 마크다운 컴포넌트를 구현하였다. 이번에는 캘린더 페이지를 구현해서 내 일정을 입력하고 캘린더로 효과적으로 보여주는 페이지를 작성할 계획이다. 또한 API를 통해서 카카오톡이나 이메일로 일정 리마인딩을 해주는 기능도 구현할 계획이다.

---

### 📖캘린더 라이브러리

캘린더 라이브러리를 통해 일정을 등록하고 관리하기 위해서는 일정을 저장할 백엔드와 데이터베이스가 필요하다. 먼저 캘린더 라이브러리는 `react-big-calendar` 를 사용할 것이다.

```ts
const MyCalendar = () => {
    const [view, setView] = useState<View>(Views.MONTH);
    const [date, setDate] = useState(new Date());

    return (
        <CalenderWrapper>
            <Calendar
                localizer={localizer}
                startAccessor="start"
                endAccessor="end"
                selectable
                view={view}
                views={[Views.MONTH, Views.WEEK, Views.DAY]}
                defaultView={Views.MONTH}
                onView={(v) => setView(v)}
                date={date}
                onNavigate={(newDate) => setDate(newDate)}
                popup
                culture="en"
                events={[]}
                messages={{
                    today: "today",
                    previous: "previous",
                    next: "next",
                    month: "month",
                    week: "week",
                    day: "day",
                    agenda: "agenda",
                    date: "date",
                    time: "time",
                    event: "event",
                    noEventsInRange: "no events",
                }}
            />
        </CalenderWrapper>
    );
};
```

react 전용 라이브러리다 보니까 State관리로 간편하게 커스터마이징이 간단하다. 일단 간단한 Month, Week, Today 달력부터 출력하는 것을 진행 하였다.

---
### 🛠️결과물

<img width="1900" height="911" alt="Image" src="https://github.com/user-attachments/assets/6578f391-90ae-4303-9be1-4c9851ba374a" />

간단하게 달력을 출력을 해보았으며, 다음번에는 캘린더에 일정을 저장하고 좌측에 출력하는 기능을 구현할 것이다. 그러기 위해서는 백엔드와 연동하여 일정을 저장해주어야 하므로 연동 또한 구현할 것이다.
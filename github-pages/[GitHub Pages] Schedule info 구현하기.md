[Velog로 가기](https://velog.io/@choi-hyk/GitHub-Page-Schedule-info-구현하기)

released at 2025-07-28 19:25:58 KST

updated at 2025-12-28 16:57:05 KST

|[github pages](https://velog.io/tags/github-pages)|
|----|

## 🛠️ Schedule info 구현하기

저번에 이어서 오늘은 캘린더의 일정을 띄워주는 **Schedule info** 컴포넌트를 구현하고 **Calender** 페이지 구현을 일단 마치고 main 브랜치에 푸시하려 한다. 원래는 GitHub Pages에서 일정을 등록하고 수정할 수 있는 기능도 넣으려 했으나, 인증 과정을 아직 구현하기에는 이르다고 생각해 간단하게 일정 정보를 출력하는 기능만 구현하였다.

---

### 구현 방법

구현은 간단하지만, 아무래도 CSS 스타일링이 오래 걸렸다. 그리고 react-big-calendar도 생소하다 보니 생각보다 시간이 걸렸다.

구현은 일단 캘린더 우측에 일정의 제목과 시간 그리고 내용을 출력하는 컴포넌트를 제작하는 것이었다.

<img width="770" height="129" alt="Image" src="https://github.com/user-attachments/assets/835fcce5-d46d-4cbe-8f17-5e86658e312b" />

기존의 `events` 데이터는 위와 같이 `description` 필드가 없어서 Docker에서 돌아가고 있는 MySQL의 `events` 테이블에 `description` 필드를 추가하였다. 그리고 테스트 일정으로 이틀 이상의 일정과 하루 3개 이상의 일정 데이터도 추가하였다.

<img width="1919" height="192" alt="Image" src="https://github.com/user-attachments/assets/b6dd820b-8689-4dfc-9ff5-64cef169622b" />

이제 `Date`를 기준으로 필터링해서 캘린더에서 선택된 날짜의 일정들을 필터링하면 된다.

#### Calender Page

```js
function Calender() {
    const [events, setEvents] = useState<Event[] | undefined>([]);
    const [selectedDate, setSelectedDate] = useState<Date>(new Date());
    const [selectedEvents, setSelectedEvents] = useState<Event[] | undefined>(
        []
    );

    useEffect(() => {
        fetchEvents().then((events) => {
            const parsed = events.map((e: Event) => ({
                ...e,
                start: new Date(e.start),
                end: new Date(e.end),
            }));
            setEvents(parsed);
        });
    }, []);

    useEffect(() => {
        if (!events) return;

        const filtered = events.filter((event) => {
            const s = event.start as Date;
            const e = event.end as Date;
            return (
                s.getFullYear() <= selectedDate.getFullYear() &&
                s.getMonth() <= selectedDate.getMonth() &&
                s.getDate() <= selectedDate.getDate() &&
                e.getFullYear() >= selectedDate.getFullYear() &&
                e.getMonth() >= selectedDate.getMonth() &&
                e.getDate() >= selectedDate.getDate()
            );
        });
        setSelectedEvents(filtered);
    }, [selectedDate, events]);

    return (
        <>
            <MarkdownRenderer page="calender" />
            <DivWrapper>
                <MyCalendar
                    events={events}
                    selectedDate={selectedDate}
                    setSelectedDate={setSelectedDate}
                />
                <EventInfo events={selectedEvents} />
            </DivWrapper>
        </>
    );
}

export default Calender;
```

`Context`로 일정을 관리할까 고민했지만 일정 객체가 Calender Page에서만 쓰이기 때문에 `Props`로 내려주었다.

#### EventInfo Component

```js
function EventInfo({ events }: { events: Event[] | undefined }) {
    if (!events || events.length === 0) {
        return null;
    }
    return (
        <DivEventsContainer>
            {events.map((ev) => (
                <DivEventInfo key={ev.id}>
                    <DivEventTitle>{ev.title}</DivEventTitle>
                    <DivEventDate>
                        <span style={{ color: "var(--secondary-text-color)" }}>
                            {moment(ev.start).format("MMM DD, YYYY HH:mm")}
                        </span>
                        <span> ~ </span>
                        <span style={{ color: "var(--tertiary-text-color)" }}>
                            {moment(ev.end).format("MMM DD, YYYY HH:mm")}
                        </span>
                    </DivEventDate>
                    <DivEventDescription>{ev.description}</DivEventDescription>
                </DivEventInfo>
            ))}
        </DivEventsContainer>
    );
}

export default EventInfo;
```

이렇게 구현은 완료하였고 CSS는 살펴보지 않겠다.~~(사실상 GPT가 구현함...)~~

---

### 최종 결과물

<img width="1912" height="908" alt="Image" src="https://github.com/user-attachments/assets/4ed70a75-7269-4c1b-8f27-8575accef774" />

위의 사진처럼 이틀 이상의 일정이 달력에 추가되고 `selectedDate`도 성공적으로 캘린더에 출력된다. 그리고 `selectedDate`로 `selectedEvents`를 필터링하여 우측에 출력되는 모습도 볼 수 있다.

---

### 마무리

일단 Calender page는 이렇게 마무리하고, 나중에 다른 페이지도 얼추 구현하면 일정 등록 로직을 구현할 생각이다.

다음번에는 study에 Velog 글을 출력해주는 기능을 구현할 예정이다. 또한 GitHub 페이지를 추가하여 내 GitHub 활동 로그를 보여주는 작업도 할 생각이다.

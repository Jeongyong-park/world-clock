<script lang="ts">
    let currentDate: Date = new Date();

    // 시간 갱신을 위한 인터벌 설정
    setInterval(() => {
        currentDate = new Date();
    }, 1000);

    // 날짜 및 시간 포매팅 함수
    const formatTimeUnit = (unit: number): string =>
        (unit < 10 ? "0" : "") + unit;

    // 날짜 및 시간 계산 함수
    const getDateParts = (date: Date, offset: number = 0) => {
        const adjustedDate = new Date(date.getTime() + offset);
        return {
            year: adjustedDate.getUTCFullYear(),
            month: formatTimeUnit(adjustedDate.getUTCMonth() + 1),
            day: formatTimeUnit(adjustedDate.getUTCDate()),
            hours: formatTimeUnit(adjustedDate.getUTCHours()),
            minutes: formatTimeUnit(adjustedDate.getUTCMinutes()),
            seconds: formatTimeUnit(adjustedDate.getUTCSeconds()),
        };
    };

    const isDaylightSavingTime = (date: Date) => {
        const year = date.getFullYear();
        // 일광 절약 시간제 시작: 3월 두 번째 일요일
        const dstStart = new Date(
            year,
            2,
            14 - (new Date(year, 2, 1).getDay() || 7),
        );
        // 일광 절약 시간제 종료: 11월 첫 번째 일요일
        const dstEnd = new Date(
            year,
            10,
            7 - (new Date(year, 10, 1).getDay() || 7),
        );

        return date >= dstStart && date < dstEnd;
    };

    interface TimeZone {
        name: string;
        offset: number | (() => number);
    }
    // 시간대 정보를 배열로 관리
    const timeZones: TimeZone[] = [
        { name: "LONDON", offset: 0 },
        { name: "SEOUL", offset: 9 * 60 * 60 * 1000 }, // 한국 표준시
        {
            name: "NEWYORK",
            get offset() {
                // EST 기본 오프셋 설정
                let offset = -5 * 60 * 60 * 1000; // EST 기본 오프셋
                // 일광 절약 시간 적용 여부 확인
                if (isDaylightSavingTime(currentDate)) {
                    offset = -4 * 60 * 60 * 1000; // DST 적용 시 EST 오프셋 조정
                }
                return offset;
            },
        },
    ];

    // 각 시간대별 날짜 및 시간 계산
    $: timeZoneDateParts = timeZones.map((zone) => ({
        name: zone.name,
        dateParts: getDateParts(
            currentDate,
            typeof zone.offset === "function" ? zone.offset() : zone.offset,
        ),
    }));

    function spaceStr(strValue: string, totalLength: number): string {
        let spaceNeeded = totalLength - strValue.length;
        let spaceStr = "";
        for (let idx = 0; idx < spaceNeeded; idx++) {
            spaceStr += "&nbsp;"; // 필요한 만큼 공백 추가
        }
        return strValue + spaceStr; // 원래 문자열에 공백 문자열 추가
    }
</script>

<div class="clock digital">
    {#each timeZoneDateParts as { name, dateParts }}
        <p>
            {@html spaceStr(name, 10)}: {dateParts.year}/{dateParts.month}/{dateParts.day}
            {dateParts.hours}:{dateParts.minutes}:{dateParts.seconds}
        </p>
    {/each}
</div>

<style>
    .clock.digital {
        font-family: "Digital Numbers", monospace;
        font-size: 2rem;
    }
</style>

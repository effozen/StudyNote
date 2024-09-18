
- 개요
	- 배경
		- 네이버 부스트캠프를 시작하고, 멤버십 미션이 주어졌다.
		- 난 왜 이렇게 시작을 했는가?
- 순수한 HTML/CSS/JS에 대한 사용에 대한 도전 계기
	- 배경을 이유 삼아서 순수한 `HTML/CSS/JS`를 사용하기로 했다.
	- 처음에 순수한 HTML/CSS/JS를 사용해서 하려고 했고 하나의 파일에 모든 내용을 암고자 했다
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>History Modal</title>
    <link rel="stylesheet" href="historyList.css">
</head>
<body>
    <article class="history-list surface--default rounded--200 shadow--16">
        <header class="history-list__header flex justify-between items-center">
            <h2 class="display-bold16 text--strong">사용자 활동 기록</h2>
            <button class="flex justify-between items-center">
                <img class="size-m" src="../../../../assets/icons/close.svg" alt="">
                <span class="display-bold14 text--history">닫기</span>
            </button>
        </header>
        <section class="history-list__item empty">
            <h2 class="history-list__item--empty display-medium14 text--weak">사용자 활동 기록이 없습니다.</h2>
        </section>
        <div class="flex flex-col">
            <section class="history-list__item flex">
                <div class="size-xl flex__item--fix rounded--500 border--default ">
                    <img class="size-full rounded--500" src="../../../../assets/imgs/test.jpg" alt="user profile">
                </div>
                <div class="history-list__item__log flex flex-col">
                    <h4 class="text--history display-medium14">UserName</h4>
                    <p class="display-medium14 text--history"><em>{title}</em>를 <em>{ColumnName}</em>에서 <em>{ColumnName}</em>으로 <em>{Action}</em>하였습니다.</p>
                    <time class="display-medium12 text--weak" datetime="2024-08-22">TimeStamp</time>
                </div>
            </section>
            <section class="history-list__item flex">
                <div class="size-xl flex__item--fix rounded--500 border--default ">
                    <img class="size-full rounded--500" src="../../../../assets/imgs/test.jpg" alt="user profile">
                </div>
                <div class="history-list__item__log flex flex-col">
                    <h4 class="text--history display-medium14">UserName</h4>
                    <p class="display-medium14 text--history"><em>{title}</em>를 <em>{ColumnName}</em>에서 <em>{ColumnName}</em>으로 <em>{Action}</em>하였습니다.</p>
                    <time class="display-medium12 text--weak" datetime="2024-08-22">TimeStamp</time>
                </div>
            </section>
            <section class="history-list__item flex">
                <div class="size-xl flex__item--fix rounded--500 border--default ">
                    <img class="size-full rounded--500" src="../../../../assets/imgs/test.jpg" alt="user profile">
                </div>
                <div class="history-list__item__log flex flex-col">
                    <h4 class="text--history display-medium14">UserName</h4>
                    <p class="display-medium14 text--history"><em>{title}</em>를 <em>{ColumnName}</em>에서 <em>{ColumnName}</em>으로 <em>{Action}</em>하였습니다.</p>
                    <time class="display-medium12 text--weak" datetime="2024-08-22">TimeStamp</time>
                </div>
            </section>
        </div>
    </article>
</body>
</html>
```
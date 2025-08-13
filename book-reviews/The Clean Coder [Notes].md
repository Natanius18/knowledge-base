## Chapter 1 Professionalism

**Be Careful What You Ask For**

**Taking Responsibility**

**First, Do No Harm**

**Work Ethic**

*\[My notes\]:* Стремиться к 100% покрытию тестами.

Если профессионал допустил баг, который стоил компании \$10к, то он выплатит компании \$10к.

Работать 40 часов на работодателя + 20 часов на себя, изучая что-то новое и практикуясь.

168 часов в неделю: 60 работа, 56 сон, 52 на все остальное.

На бумаге выглядит хорошо, но по факту так не выходит.

Начиная новый проект, прочитай пару книг о домене, поговори с экспертами, проинтервьюируй кастомера и юзеров.

## Chapter 2 Saying No

**Adversarial Roles**

**High Stakes**

**Being a “Team Player”**

**The Cost of Saying Yes**

**Code Impossible**

*\[My notes\]:* никаких "я постараюсь". Уметь говорить нет. Не пытаться быть героем. Глава больше для менеджеров.

## Chapter 3 Saying Yes

**A Language of Commitment**

**Learning How to Say “Yes”**

*\[My notes\]:* "Say. Mean. Do." Снова никаких "я постараюсь". When professionals say yes, they use the language of commitment so that there is no doubt about what they’ve promised.

## Chapter 4 Coding

**Preparedness**

**The Flow Zone**

**Writer’s Block**

**Debugging**

**Pacing Yourself**

**Being Late**

**Help**

*\[My notes\]:* When you cannot concentrate and focus sufficiently, the code you write will be wrong.

The worst code I ever wrote was at 3 am.

Don’t write code when you are tired.

When I am worried about an argument with my wife, or a customer crisis, or a sick child, I can’t maintain focus. My concentration waversю This is no time to code. Any code I produce will be trash. So instead of coding, I need to resolve the worry.

The Zone is not where you want to be. (Это типа состояние потока, которое мега продуктивное, когда ты в максимальном фокусе, но он утверждает, что это лишь иллюзия и на самом деле потом возвращаться и переделывать).

Музыку слушать нельзя, потому что это мешает фокусироваться.

Если тебя отвлекают, то это плохо. Предлагает решения: парное программирование, TDD. Партнер или тесты будут держать контекст и ты сможешь вернуться.

When you are stuck, when you are tired, disengage for awhile.

Sometimes the best way to solve a problem is to go home, eat dinner, watch TV, go to bed, and then wake up the next morning and take a shower.

Do not hope that you can get it all done in Х days!

You should not agree to work overtime unless (1) you can personally afford it, (2) it is short term, two weeks or less, and (3) your boss has a fall-back plan in case the overtime effort fails.

Define “DONE”.

Проси помощи.

Предлагай помощь.

Принимай помощь.

Очень похоже что чувак никогда не работал, потому что according to this я бы ни строчки кода в жизни не написал. А реально — когда работать? Только в паре, получается, потому что парное программирование не заведет в поток по его словам и поможет восстановить контекст если тебя отвлекли. Так что, теперь нанимать разработчиков только парами? А может перестать задалбывать их митингами?

## Chapter 5 Test Driven Development

**The Jury Is In**

**The Three Laws of TDD**

**What TDD Is Not**

1. You are not allowed to write any production code until you have first written a failing unit test.

2. You are not allowed to write more of a unit test than is sufficient to fail — and not compiling is failing.

3. You are not allowed to write more production code that is sufficient to pass the currently failing unit test.

30 секунд на круг, писать тест потом часть кода, чтобы тест прошел.

Это поможет правильному дизайну.

## Chapter 6 Practicing

**Some Background on Practicing**

**The Coding Dojo**

**Broadening Your Experience**

*\[My notes\]:* Нужно практиковаться, чтобы выработать автоматические решения задач, чтобы потом реагировать быстро и эффективно. Но для практики советует выбирать новый язык программирования, а не тот, на котором работаешь, чтобы развивать свои навыки полиглота. Роналду наверное потому и хорош, что гонял с пацанами в баскет, бейсбол и теннис между своими матчами.

## Chapter 7 Acceptance Testing

**Communicating Requirements**

**Acceptance Tests**

*\[My notes\]:* Стейкхолдеры сами не знают чего хотят, поэтому чем точнее требования — тем больше вероятность, что они захотят другое в конце. И девелоперы не должны давать четких эстимейтов, на то они и эстимейты. Чем позже — тем точнее.

Хотя от нас почему-то требуют четких эстимейтов, не давая при этом всех требований...

Также достопочтенный Роберт утверждает, что это обязанность девелопера инициировать встречи со стейкхолдерам и тестировщиками, чтобы обсудить требования, составить acceptance tests и убедиться, что все понимают что будет в итоге.

Как по мне, пока мне платят почасово, а не за выкаченную в продакшн фичу, то это только в интересах стейкхолдера получить эту фичу именно так как они хотели как можно быстрее. Конечно, если у меня есть сомнения в понимании каких-то требований, то я уточню, но если я уверен что мне все понятно и при этом сделал не то что они имели в виду, то это только их проблема. Я то смогу переделать сколько угодно раз. Мне за время платят.

**Definition of done.**

Done means done.

Done means all code written, all tests pass, QA and the stakeholders have accepted. Done.

Acceptance tests should always be automated.

In an ideal world, the stakeholders and QA would collaborate to write these tests, and developers would review them for consistency. If it turns out that developers must write these tests, then take care that the developer who writes the test is not the same as the developer who implements the tested feature.

Typically business analysts write the “happy path” versions of the tests, because those tests describe the features that have business value. QA typically writes the “unhappy path” tests, the boundary conditions, exceptions, and corner cases.

А у нас все вообще не так, девелоперы сами пишут тесты к своим же фичам.

It is the developer’s job to connect the acceptance tests to the system, and then to make those tests pass.

Я видел такие рукожопые имлементации кукумбер тестов, что даже пьяным такое сложно придумать. Хотя сами сценарии выгялдили вполне нормально. Зато все тесты проходили и считалось что фича готова, хотя по факту тесты ничего не тестировали.

It is much better to write a test that selects the button whose ID is ok_button than it is to select the button in column 3 of row 4 of the control grid.

А нам правда нужны такие советы от великого гуру разработки? Ну я же не один считаю это настолько естественно и очевидно, что по-другому как будто бы даже и нельзя?

It is very important to keep the CI tests running at all times. They should never fail. If they fail, then the whole team should stop what they are doing and focus on getting the broken tests to pass again. A broken build in the CI system should be viewed as an emergency, a “stop the presses” event.

Ну это перебор. Кто накосячил тот починет. Чего все должны бросать свою работу? Каждый в своей ветке и все норм. Про постоянные билды после каждого коммита это хорошая практика, да. Хотя вроде как это у многих по умолчанию: либо дженкинс, либо гитлаб пайплайны, но вроде как и не самый очевидный совет, так что ладно, зачтем.

## Chapter 8 Testing Strategies

**QA Should Find Nothing**

**The Test Automation Pyramid**

*\[My notes\]:* Component tests are written by QA and Business with assistance from development. They are composed in a component-testing environment such as FitNesse, JBehave, or Cucumber. (GUI components are tested with GUI testing environments such as Selenium or Watir.) The intent is that the business should be able to read and interpret these tests, if not author them. Ну снова то же самое. В нашем случае бизнес понятия не имеет как фичи должны работать, о каком понимании тестов вообще может быть речь?

## Chapter 9 Time Management

**Meetings**

**Focus-Manna**

**Time Boxing and Tomatoes**

**Avoidance**

**Blind Alleys**

**Marshes, Bogs, Swamps, and Other Messes**

*\[My notes\]:* Meetings cost about \$200 per hour per attendee. This takes into account salaries, benefits, facilities costs, and so forth. Чувак хочет сказать, что у него в команде все зарабатывают \$33600 в месяц?

There are two truths about meeting.

1. Meetings are necessary.

2. Meetings are huge time wasters.

Often these two truths equally describe the same meeting. Ну это клиника уже мне кажется...

One of the most important duties of your manager is to keep you out of meetings. A good manager will be more than willing to defend your decision to decline attendance because that manager is just as concerned about your time as you are. Снова чтиво для менеджеров. Нашим точно бы не помешало почитать.

От бесполезных митингов надо отказываться, со скучных митингов надо уходить: remaining in a meeting that has become a waste of time for you, and to which you can no longer significantly contribute, is unprofessional. You have an obligation to wisely spend your employer’s time and money, so it is not unprofessional to choose an appropriate moment to negotiate your exit. Всегда узнавать агенду митинга и цели.

How do you get the data you need to settle a disagreement? Sometimes you can run experiments, or do some simulation or modeling. But sometimes the best alternative is to simply flip a coin to choose one of the two paths in question. If things work out, then that path was workable. If you get into trouble, you can back out and go down the other path. Серьезно? Это совет профессионала? Бросить монетку?

I have found that once the manna is gone, you can’t force the focus. You can still write code, but you’ll almost certainly have to rewrite it the next day, or live with a rotting mass for weeks or months. So it’s better to take thirty, or even sixty minutes to de-focus. С таким подходом, как я уже говорил ранее, я бы писал код 0 часов в жизни. Ну может 10 в сумме, ладно...

Nothing has a more profound or long-lasting negative effect on the productivity of a software team than a mess. Nothing. Скажите это менеджерам плиз. Мы и так знаем.

Moving forward through a swamp, when you know it’s a swamp, is the worst kind of priority inversion. By moving forward you are lying to yourself, lying to your team, lying to your company, and lying to your customers. You are telling them that all will be well, when in fact you are heading to a shared doom. Опять же, на моей практике команда всегда говорит об этом. Менеджеры решают, что это неважно и форсят пилить фичи вместо рефакторинга.

## Chapter 10 Estimation

**What Is an Estimate?**

**PERT**

**Estimating Tasks**

**The Law of Large Numbers**

*\[My notes\]:* В принципе то же самое, что уже было сказано во второй главе, эстимейт это не коммитмент + подробнее про техники эстимирования, типа скрам покер. Прикольная система этот PERT, жаль только что никто не пользуется.

## Chapter 11 Pressure

**Avoiding Pressure**

**Handling Pressure**

*\[My notes\]:* Чувак с опытом 20 лет повелся на какой-то дурацкий стартап, где только спустя 4 года (!) очень стрессовой безрезультативной работы (по ночам и по выходным в том числе), после того как ему жена сказала, что он идиот, только о работе и думает, а дети растут без отца, — понял, что он действительно идиот; уволился, стал консультантом и теперь капец как гордится тем, что больше не называет никого "босс". Ну реально идиот.

The best way to stay calm under pressure is to avoid the situations that cause pressure. То есть не работать в айти, правильно? Ну анекдот.

Choose disciplines that you feel comfortable following in a crisis. Then follow them all the time. Following these disciplines is the best way to avoid getting into a crisis. Звучит как что-то мудрое. Только обычно на проектах уже все решено, ты не выбираешь чему следовать.

So I want you sitting around tables facing each other. I want you to be able to smell each other’s fear. Тут в нем прям какой-то сильный писатель проснулся.

## Chapter 12 Collaboration

**Programmers versus People**

**Cerebellums**

*\[My notes\]:* Сначала повыпендривался, что они с другом в гараже сделали что-то мега крутое. При этом у них была одна проблемка и они решили ее пофиксить. Tim and I were not experts on data structures and algorithms. We’d never heard of hash tables or binary searches. We had no clue how to make an algorithm fast. We just knew that what we were doing was too slow. Хоть бы постеснялся такое говорить.

In the end we got the time down under 15 minutes, which was very close to how long it took simply to read the source tape. So we were satisfied. Короче потратили кучу времени и в итоге вышли в 0 и were satisfied.

Потом очередная крутая история о том, как его уволили за то что он явился без галстука и постоянно опаздывал.

I went home that day to my pregnant 22-year-old wife and had to tell her that I’d been fired. That’s not an experience I ever want to repeat. Молодец.

I want the team to own the code, not the individuals. Типа чтоб никто не строил барьеров вокруг своего кода, делился с тиммейтами и все шарили за всё. Мне одному это кажется логично и правильно без всяких книжек?

Рассказывает, что объединятся это всегда хорошо. Согласен, если у тебя есть адекватные люди, с которыми можно объединяться, а не кучка индусов, которые не отстреливают, что происходит.

Perhaps we didn’t get into programming to work with people. Tough luck for us. Programming is all about working with people. We need to work with our business, and we need to work with each other. Ну тут, к сожалению, правда.

## Chapter 13 Teams and Projects

**Does It Blend?**

*\[My notes\]:*  Now here’s a rule: There is no such thing as half a person. It makes no sense to tell a programer to devote half their time to project A and the rest of their time to project B, especially when the two projects have two different project managers, different business analysts, different programmers, and different testers. Полностью согласен, но разве это не очевидно? Очевидно. Но только не для менеджеров почему-то.

So a nicely gelled team of twelve might have seven programmers, two testers, two analysts, and a project manager. Тут он загнул конечно. Лучшая тима на моей практике была такая: 2-3 прогера, один из которых был архитектором как бы, но сам нередко писал код, 1 тестировщик шарящий в домене и всё. Был еще ПО, но он не лез, просто раз в какое-то время мы рассказывали ему что мы сделали и какие дальнейшие планы. Идеально.

It takes time for a team like this to work out their differences, come to terms with each other, and really gel. It might take six months. It might even take a year. But once it happens, it’s magic. A gelled team will plan together, solve problems together, face issues together, and get things done. У нас люди менялись намного чаще. А в той маленькой тиме потребовалось буквально две недели чтоб все стали понимать друг друга с полуслова.

A gelled team can accept many projects simultaneously and will divvy up the work according to their own opinions, skills, and abilities. Так ты ж только что говорил что нельзя делить людей на несколько проектов...

Management can set targets for each project given to a team. For example, if the average velocity of a team is 50 and they have three projects they are working on, then management can ask the team to split their effort into 15, 15, and 20.

Aside from having a gelled team working on your projects, the advantage of this scheme is that in an emergency the business can say, “Project B is in crisis; put 100% of your effort on that project for the next three weeks.”

Reallocating priorities that quickly is virtually impossible with the teams that came out of the blender, but gelled teams that are working on two or three projects concurrently can turn on a dime.

Опять то же самое. Так можно делить людей или нет?

Teams are harder to build than projects. Therefore, it is better to form persistent teams that move together from one project to the next and can take on more than one project at a time. The goal in forming a team is to give that team enough time to gel, and then keep it together as an engine for getting many projects done. Ну с выводами в принципе согласен. Очередная фантастическая глава для менеджеров.

## Chapter 14 Mentoring, Apprenticeship, and Craftsmanship

**Degrees of Failure**

**Mentoring**

**Apprenticeship**

**Craftsmanship**

Чувак был разочарован девочкой, которая сказала, что не учила программирование на CS в универе и не проходила никакие курсы. What you learn in school and what you find on the job are often very different things. Тоже мне открытие. Желаю ему удачи менять систему образования.

Дальше рассказывает истории из своего детства каким умным ребенком он был и как учился программированию читая мануалы и просто наблюдая за людьми. Но говорит, что конечно было бы лучше иметь настоящего ментора.

Сравнил с докторами, рассказывает как важна стажировка в окружении синиоров.

Рассказывает про Masters, Journeymen и Apprentices/Interns.

Apprenticeship ought to last a year.

Again, all of this is idealized and hypothetical. Спасибо, что уточнил.

The difference is the very notion that professional values and technical acumen must be taught, nurtured, nourished, coddled, and encultured. What’s missing from our current sterile approach is the responsibility of the elders to teach the young. Боже сколько лишних слов.

Craftsmanship is a meme that contains values, disciplines, techniques, attitudes, and answers. Вот только мемов нам тут не хватало.

It is time for those of us in the software industry to face the fact that guiding the next batch of software developers to maturity will fall to us, not to the universities. А, он не собирается менять систему образования — решил, что сам научит как надо. Ладно.

## Appendix A Tooling

**Tools**

**Source Code Control**

**IDE/Editor**

**Issue Tracking**

**Continuous Build**

**Unit Testing Tools**

**Component Testing Tools**

**Integration Testing Tools**

**UML/MDA**

*\[My notes\]:* напоследок накидал список всем известных тулов, которыми он пользуется.


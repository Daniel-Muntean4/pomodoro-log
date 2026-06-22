# Svelte + Vite - Pomodoro Website

This app allow students to organize their studies with fixed time of work/study that are registered.

The entities are the study sessions which have a date, duration and finish informations. As well as topic, or subject for which I the user will study.

The application  has a light dark/version. The events are stored in browser local storage. Framework: Svelte.


## Flows of the app

This website allows the user to set a pomodoro timer, choosing from the 25, 30 and 50 minutes options, also the user should press a box from the options for topics available, that option will be considered the next session topic. The user will be able to choose the topic by clicking, not writing the topics.

The moment start is pressed teh session starts, when the user presses stop the session ends and is recorded in history, if the user finished the session without pausing the session is recorded at the end.

The moment the active session ends the short pause starts, after three short pauses the long pauses start instead.
The pomodoro, short and long breaks can be accessed by buttons by the user, as well.

When pressing History a schedule history is displayed, that can be filtered by date, week, month etc. Also the history can be filtered by topic or by combining date and topic.

The history and the prefered theme are saved after the user exiting the site.

## Backend features
Information is saved in a H2 database not in the localStorage, if the server is available, and pagination allows navigation with 12 elements per page.

## Demo:

<img width="800" height="518" alt="Grabación de pantalla 2026-06-11 a las 17 19 43" src="https://github.com/user-attachments/assets/6dfd8748-684d-4378-a2da-b6ccabc8a2af" />

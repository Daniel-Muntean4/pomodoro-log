
<script>
		const stopTime = 1000000*100000000;
		const DAY = 24*60*60*1000;
		let pomodoroMinutes =$state(25) ;
		let breakMinutes =$state(5);
	const loadedData = localStorage.getItem('pomodoro-log-saved-sessions');
	const loadedValues = loadedData ? JSON.parse(loadedData) : [];
    let elapsed = $state(pomodoroMinutes*60);
		let elapsedBreak= $state(breakMinutes*60);
 let topicFilter = $state('all')
		let dateFilter = $state('all')
    let interval = $state(stopTime);
 		const savedTheme = localStorage.getItem('pomodoro-log-theme');
		 const loadedTheme = savedTheme ? JSON.parse(savedTheme) : 'light';
		let theme = $state(loadedTheme);
		let timeMinutes= $derived(Math.floor(elapsed/60));
		let timeSeconds= $derived(elapsed%60);
		let breakTimeMinutes= $derived(Math.floor(elapsedBreak/60));
		let breakTimeSeconds= $derived(elapsedBreak%60);
		let restState = $state(false);

		let restsCounter = $state(3);
		let topics = new Map();
		let firstTopic =  $state("Java Backend");
		let secondTopic =  $state("Javascript Frontend");
		let thirdTopic =  $state("Electronic Circuits");
		let fourthTopic =  $state("Embedded Systems");
		topics.set(1,firstTopic);
		topics.set(2,secondTopic);
		topics.set(3,thirdTopic);
		topics.set(4,fourthTopic);
		let currentTopicText=$state("Java Backend");
		let currentTopic = $state(1)
		let sessionLogs= $state(loadedValues);
		let filteredSessions= $derived(sessionLogs.filter(log=>log[3]==topicFilter));
		let filteredByDateSessions= $derived(sessionLogs.filter(log=>Date.now() - log[0] <= DAY*dateFilter));
		let filteredByDateAndTopicSessions= $derived(filteredSessions.filter(log=>Date.now() - log[0] <= DAY*dateFilter));

		let timeStart= $state(0);
		let timeStop= $state(0);
		let isHistoryMode = $state(false)


	function setTopic(topicId) {
     	currentTopic=topicId;
			currentTopicText=topics.get(topicId);
	
     }
	function setStart() {
     	interval = 1000;
		timeStart=Date.now();

     }
	function setStop() {
		interval = stopTime;
		saveSession();

	}
	function saveSession(){
		timeStop = Date.now();
		let oneSessionDuration = timeStop - timeStart;
		sessionLogs.push([timeStart, timeStop, oneSessionDuration, currentTopic]);
		localStorage.setItem('pomodoro-log-saved-sessions', JSON.stringify(sessionLogs));

	}
		function setPomodoro(minutes) {
     	restState=false;
		 pomodoroMinutes = minutes;
			 elapsed=pomodoroMinutes*60;
			 
     }
	
	function setShortRest() {
     	     	restState=true;
				  breakMinutes=5;
		elapsedBreak = breakMinutes*60;


     }function setLongRest() {
     	     restState=true;
						breakMinutes = 10;
						restsCounter=3;
			elapsedBreak = breakMinutes*60;

     }

		$effect(() => {
			document.documentElement.dataset.theme=theme;}
		);
		$effect(() => {
			localStorage.setItem('pomodoro-log-theme', JSON.stringify(theme));
				}
		);
			$effect(() => {
        const id = setInterval(() => {

			if(restState===false){
						elapsed -= 1;
				if (elapsed===0) {
					 saveSession();
					if (restsCounter === 0) {
						setLongRest();

					} else {
						setShortRest();
						restsCounter-=1;

					}


				}

					}
					else {
				elapsedBreak -= 1;


				if (elapsedBreak===0){
					restState=false;
					elapsed=pomodoroMinutes*60;
					timeStart=Date.now();
				}
			}


        }, interval);
        return () => {
            clearInterval(id);
        };
    });

</script>
<p class="toggle">
<button class="toggle-item"  onclick={()=> theme = theme === 'dark' ? 'light' : 'dark' }>
	{(theme === 'dark') ? "🌙 ": "☀️"}
</button>
</p>
{#if isHistoryMode!==true}
<h2>Timer</h2>
	<br>

	<p class="inline-btn ">

<button onclick={() => setPomodoro(pomodoroMinutes)}>
	Pomodoro
</button>

<button  onclick={setShortRest} >
	Short Break
</button>

<button onclick={setLongRest}  >
	Long Break
</button>
</p>
	<br>
	<br>

	{#if restState===false}

<p class="clock">{timeMinutes}:{timeSeconds<=9 ? '0' : ''}{timeSeconds}</p>

		<br>

{:else}
<p class="clock">{breakTimeMinutes>9 ? '' : '0'}{breakTimeMinutes}:{breakTimeSeconds<=9 ? '0' : ''}{breakTimeSeconds}</p>
		<br>

{/if}
	<br>
<p  class="start-stop">
	<button  onclick={setStart} >
	Start
</button>
<button  onclick={setStop } >
	Pause
</button>
</p>
<p>
	<br>

	<input  type="radio"
	        value={25}
	        bind:group={pomodoroMinutes}

	        onchange={()=>setPomodoro(pomodoroMinutes)}
	> 25 min
<input type="radio"
       name = "time"
	   value={30}
		bind:group={pomodoroMinutes}
       onchange={()=>setPomodoro(pomodoroMinutes)}
	> 30 min
<input type="radio"
       value={50}
       bind:group={pomodoroMinutes}
       onchange={()=>setPomodoro(pomodoroMinutes)}
	> 50 min
	</p>

	<br>

<p class="topics">
<button class="topic-btn" onclick={()=>setTopic(1)}>
	{firstTopic}
</button>
<button class="topic-btn"  onclick={()=>setTopic(2)} >
	{secondTopic}
</button>
</p>
	<p class="topics">

	<button class="topic-btn"  onclick={()=>setTopic(3)} >
	{thirdTopic}
</button>
<button class="topic-btn" onclick={()=>setTopic(4)}   >
	{fourthTopic}
</button>
	</p>

{:else}

	<h2>History</h2>
	<select
	bind:value={topicFilter}>
		<option value='all'>all</option>
		<option value='1'>{firstTopic}</option>
		<option value='2'>{secondTopic}</option>
		<option value='3'>{thirdTopic}</option>
		<option value='4'>{fourthTopic}</option>

	</select>
	<select
			bind:value={dateFilter}>
		<option value='all'>all</option>
		<option value={1}>Last Day</option>
		<option value={7}>Last Week</option>
		<option value={30}>Last 30 Days</option>

	</select>
	{#if topicFilter ==='all' && dateFilter==='all'}

	{#each sessionLogs as s}

		<p>Date: {new Date( s[0]).toLocaleDateString()} Start: {new Date( s[0]).toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' ,second: '2-digit'})} Finish:  {new Date( s[1]).toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' ,second: '2-digit'})} Topic: {topics.get(s[3])} Duration: {Math.floor(s[2]/60000)} minutes
		</p>{/each}

	{:else if topicFilter ==='all' && dateFilter!=='all'}
		{#each filteredByDateSessions as s}

		<p>Date: {new Date( s[0]).toLocaleDateString()} Start: {new Date( s[0]).toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' ,second: '2-digit'})} Finish:  {new Date( s[1]).toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' ,second: '2-digit'})} Topic: {topics.get(s[3])} Duration: {Math.floor(s[2]/60000)} minutes
		</p>{/each}
	{:else if topicFilter !=='all' && dateFilter==='all'}
		{#each filteredSessions as s}

			<p>Date: {new Date( s[0]).toLocaleDateString()} Start: {new Date( s[0]).toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' ,second: '2-digit'})} Finish:  {new Date( s[1]).toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' ,second: '2-digit'})} Topic: {topics.get(s[3])} Duration: {Math.floor(s[2]/60000)} minutes
			</p>{/each}
	{:else }

		{#each filteredByDateAndTopicSessions as s}
		<p>Date: {new Date( s[0]).toLocaleDateString()} Start: {new Date( s[0]).toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' ,second: '2-digit'})} Finish:  {new Date( s[1]).toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' ,second: '2-digit'})} Topic: {topics.get(s[3])} Duration: {Math.floor(s[2]/60000)} minutes
		</p>{/each}



	{/if}

	{#if sessionLogs.length ===0}
		empty list
	{/if}

{/if}
<br>
<button onclick={()=>{isHistoryMode= !isHistoryMode}}  >
 {!isHistoryMode ? 'History' : 'Timer'}

</button>

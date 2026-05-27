
<script>

		let pomodoroMinutes =$state(25) ;
	const loadedData = localStorage.getItem('pomodoro-log-saved-sessions');
	const loadedValues = loadedData ? JSON.parse(loadedData) : [];
    let elapsed = $state(25*60);
		let elapsedBreak= $state(5*60);
 let topicFilter = $derived('all')
    let interval = $state(1000000*100000000);
 		const savedTheme = localStorage.getItem('2');
		 // const loadedTheme = savedTheme ? JSON.parse(savedTheme) : 'light';
		 // console.log(savedTheme)
		let theme = $state((savedTheme) ? savedTheme : 'light');
		let timeMinutes= $derived(Math.floor(elapsed/60));
		let timeSeconds= $derived(elapsed%60);
		let breakTimeMinutes= $derived(Math.floor(elapsedBreak/60));
		let breakTimeSeconds= $derived(elapsedBreak%60);
		let timerInformation = $state('sdf');
		let restState = $state(false);
	
		let restsCounter = $state(3);
		let topics = new Map();
		topics.set(1,"Java Backend");
		topics.set(2,"Javascript Frontend");
		topics.set(3,"Electronic Circuits");
		topics.set(4,"Arduino C/C++");
		let currentTopicText=$state("Java");

		let sessionLogs= $state(loadedValues);
		let filteredSessions= $derived(sessionLogs.filter(log=>log[3]==topicFilter));
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
		interval = 1000000 * 100000000;
		timeStop = Date.now();
		let oneSessionDuration = timeStop - timeStart;
		sessionLogs.push([timeStart, timeStop, oneSessionDuration, currentTopic]);
		localStorage.setItem('1', JSON.stringify(sessionLogs));

	}
		function setPomodoro() {
     	restState=false;
			 elapsed=25*60;
			 
     }
	
	function setShortRest() {
     	     	restState=true;
						elapsedBreak=5*60;


     }function setLongRest() {
     	     restState=true;
						elapsedBreak=10*60;

     }

		$effect(() => {
			document.documentElement.dataset.theme=theme;}
		);
		console.log(theme)
			$effect(() => {
        const id = setInterval(() => {

			if(restState===false){
						elapsed -= 1;

					}
					else{
						elapsedBreak-=1;
					}
					if (elapsed===0) {
						restsCounter-=1;
						if(restState===false){
							timeStop=Date.now()
							let oneSessionDuration=timeStop-timeStart;
						sessionLogs.push([timeStart, timeStop, oneSessionDuration, currentTopic]);
							localStorage.setItem('pomodoro-log-saved-sessions', JSON.stringify(sessionLogs));
							localStorage.setItem('2', JSON.stringify(theme));
}
						if (restsCounter===0) {
							setLongRest();
						}
						else{
							setShortRest();
							
						}
					}
					if (restState===true && elapsedBreak===0){
						restState=false;
						elapsed=25*60;
						timeStart=Date.now();

					}
        }, interval);
        return () => {
            clearInterval(id);
        };
    });

</script>
<button onclick={()=> theme = theme === 'dark' ? 'light' : 'dark' }>
	Theme
</button>
{#if isHistoryMode!==true}
<h2>Timer</h2>

<button onclick={setPomodoro}>
	Pomodoro
</button>
<button  onclick={setShortRest} >
	Short Break
</button>

<button onclick={setLongRest}  >
	Long Break
</button>
{#if restState===false}

<p >{timeMinutes}:{timeSeconds<=9 ? '0' : ''}{timeSeconds}</p>


{:else}
<p >{breakTimeMinutes}:{breakTimeSeconds<=9 ? '0' : ''}{breakTimeSeconds}</p>

{/if}
<button  onclick={setStart} >
	Start
</button>
<button  onclick={setStop } >
	Pause
</button>

<p>
<input  type="radio"
	bind:group={elapsed}
	value={25*60}
	> 25 min
<input type="radio"
		bind:group={elapsed}
	value={30*60}> 30 min
<input type="radio"
		bind:group={elapsed}
		pomodoroMinutes = 50
	value={50*60}> 50 min
	</p>



<button onclick={()=>setTopic(1)}>
	Java Backend
</button>
<button  onclick={()=>setTopic(2)} >
	Javascript Frontend
</button>

<button onclick={()=>setTopic(3)} >
	Electronic Circuits
</button>
<button onclick={()=>setTopic(4)}   >
	Embedded Systems
</button>

{:else}

	<h2>History</h2>
	<select
	bind:value={topicFilter}>
		<option value='all'>all</option>
		<option value='1'>Java</option>
		<option value='2'>JavaScript</option>
		<option value='3'>Embedded Systems</option>
		<option value='4'>Electronic Circuits</option>

	</select>
	{topicFilter}
	{#if topicFilter ==='all'}

	{#each sessionLogs as s}

		<p>Date: {new Date( s[0]).toLocaleDateString()} Start: {new Date( s[0]).toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' ,second: '2-digit'})} Finish:  {new Date( s[1]).toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' ,second: '2-digit'})} Topic: {topics.get(s[3])} Duration: {Math.floor(s[2]/60000)} minutes
		</p>{/each}

	{:else }
		{#each filteredSessions as s}

		<p>Date: {new Date( s[0]).toLocaleDateString()} Start: {new Date( s[0]).toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' ,second: '2-digit'})} Finish:  {new Date( s[1]).toLocaleTimeString('en-GB', { hour: '2-digit', minute: '2-digit' ,second: '2-digit'})} Topic: {topics.get(s[3])} Duration: {Math.floor(s[2]/60000)} minutes
		</p>{/each}

	{/if}

	{#if sessionLogs.length ===0}
	{/if}

{/if}
<button onclick={()=>{isHistoryMode= !isHistoryMode}}  >
 {!isHistoryMode ? 'History' : 'Timer'}
</button>
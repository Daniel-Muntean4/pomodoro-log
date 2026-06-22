
<script>
		let token = $state(null);
		const API = "http://localhost:8080"
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
		let isHistoryMode = $state(false);
		let currentPage = $state(0);
		let totalPages = $state(0);

		async function getToken(){
			const res = await fetch(`${API}/token`,
					{
						method: 'POST',
						headers: {'Content-type' : 'application/json'},
						body: JSON.stringify(
								{subject: 'Daniel', permissions: ['READ', 'WRITE']})
					});
			const data = await res.json();
			token = data.token;

			}
		function nextPage(){
			currentPage = currentPage+1;
			if(currentPage>totalPages-1){
				currentPage=currentPage-1;
			}

		}
		function previousPage(){
			currentPage = currentPage-1;
			if(currentPage<0){
				currentPage=0;
			}
		}
		async function loadSessions(){
			let params = new URLSearchParams({page: currentPage, size:'15'});
			if(topicFilter!=='all') params.set('topicId', topicFilter);
			if(dateFilter!=='all') params.set('after', String(Date.now()-DAY*dateFilter));

			const res = await fetchAPI(`/api/sessions?${params}`);
			if (res.ok){
				let data = await res.json();
				sessionLogs = data.content.map(s => [s.startTime, s.stopTime, s.duration, s.topicId]);
				totalPages = data.totalPages;
				if(currentPage>data.totalPages-1){
					currentPage = Math.max(0,data.totalPages-1);
				}
			}
		}
		async function fetchAPI(path, options={}) {
			if(!token) await getToken();
			const save = ()=>
				 fetch(`${API}${path}`,
					{
						...options,
						headers: {
							...(options.headers ?? {}),
						'Content-type': 'application/json',
						'Authorization': `Bearer ${token}`
					}
			});
			let res = await save();
			if(res.status===401){
				await getToken();
				res = await save();
			}
			return res;
		}


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
		async function saveSession(){
			timeStop = Date.now();
			let duration = timeStop - timeStart;
			const body = {
				startTime: timeStart,
				stopTime: timeStop,
				duration: duration,
				topicId: currentTopic
			}
			try{
				let res = await fetchAPI('/api/sessions', {
					method: 'POST',
					body: JSON.stringify(body)
						}
				);
				if(!res.ok) {
					throw new Error(`Backend rejected ${res.status}`);
				}
			} catch (e) {
				sessionLogs.push([timeStart,timeStop,duration,currentTopic]);
				localStorage.setItem('pomodoro-log-saved-sessions',JSON.stringify(sessionLogs))
			}

		}
		// function saveSession(){
		// 	timeStop = Date.now();
		// 	let oneSessionDuration = timeStop - timeStart;
		// 	sessionLogs.push([timeStart, timeStop, oneSessionDuration, currentTopic]);
		// 	localStorage.setItem('pomodoro-log-saved-sessions', JSON.stringify(sessionLogs));
		//
		// }
		function setPomodoro(minutes) {
     	restState=false;
		 pomodoroMinutes = minutes;
			 elapsed=pomodoroMinutes*60;
			 
     }
	
		function setShortRest() {
					restState=true;
					  breakMinutes=5;
			elapsedBreak = breakMinutes*60;
		}
		function setLongRest() {
				 restState=true;
							breakMinutes = 10;
							restsCounter=3;
				elapsedBreak = breakMinutes*60;
		 }

		$effect(() => {
			document.documentElement.dataset.theme=theme;}
		);
		$effect(() => {
			if(isHistoryMode) loadSessions();
		}
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
	{(theme === 'dark') ? "☀️" : "🌙" }
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
	bind:value={topicFilter} onclick={()=>currentPage=0}>
		<option value='all'>all</option>
		<option value='1'>{firstTopic}</option>
		<option value='2'>{secondTopic}</option>
		<option value='3'>{thirdTopic}</option>
		<option value='4'>{fourthTopic}</option>

	</select>
	<select
			bind:value={dateFilter} onclick={()=>currentPage=0}>
		<option value='all'>all</option>
		<option value={1}>Last Day</option>
		<option value={7}>Last Week</option>
		<option value={30}>Last 30 Days</option>

	</select>
	<div class="history-list">
<br>
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
	</div>

	{#if sessionLogs.length ===0}


	{/if}
	<p class="nav-btn">
		<br>
		<br>

		<button onclick={previousPage}  disabled= {currentPage===0}>⬅</button>
Page {(totalPages>0) ? currentPage+1 : currentPage} of {totalPages}
		<button onclick={nextPage}  disabled = {currentPage>=totalPages-1}>➡</button>
	</p>
{/if}
<p class="mode">
<button onclick={()=>{isHistoryMode= !isHistoryMode}}  >
 {!isHistoryMode ? 'History' : 'Timer'}

</button>
</p>

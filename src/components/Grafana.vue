<template>

<div class="grafana">
	<br>
	<iframe src="http://localhost:3000/dashboard-solo/db/tsimulus?orgId=1&panelId=1&tab=display&from=now-1h&to=now&refresh=1s" width="1000" height="250" frameborder="0"></iframe>
	<iframe src="http://localhost:3000/dashboard-solo/db/tsimulus?orgId=1&panelId=2&from=now-1h&to=now&refresh=1s" width="1000" height="250" frameborder="0"></iframe>
</div>
</template>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 15px;
}

a {
  color: #42b983;
}
</style>

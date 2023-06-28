<script>
	export let title = '';
  </script>
  
  <style>
	.header {
	  display: flex;
	  align-items: center;
	  padding: 16px;
	  background-color: #f0f0f0;
	}
  
	.title {
	  font-size: 24px;
	  font-weight: bold;
	  margin-right: 16px;
	}
	.contact-info {
        font-size: 10px;
        line-height: 0;

		padding-left: 10%;
    }
	.contact {
		display: flex;
		padding-left: 50%;
	}
  </style>
  
  <div class="header">
	<h1 class="title">{title}</h1>
	<div class="contact">
		<div class="contact-info">
			<p>Camila Labarca</p>
			<a href="https://github.com/camilabarca/CommBoard-Analysis/tree/master">Github repository</a>
			<p>camila.labarca@ug.uchile.cl</p>
		</div>
		<div class="contact-info">
			<p>What did you think of the page? Answer our form:</p>
			<a href="https://docs.google.com/forms/d/e/1FAIpQLSdLVoKteYsTsFjBm_BETwsK-yXClOTe5_jTh6Nf1wKVqiTSvA/viewform?usp=sf_link">Click here!</a>
		</div>
	</div>
	
  </div>
  
  
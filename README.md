
<!DOCTYPE html>
<head>
	<meta name="viewport" content="width=device-width, initial-scale=1">
	<meta charset="UTF-8">
	<title>Block Co-ordinate Descent</title>
	<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bulma/0.7.4/css/bulma.min.css">
	<link href="https://fonts.googleapis.com/css?family=Dosis" rel="stylesheet">
	<style>
		* {
			font-family: 'Dosis', sans-serif;
		}
	</style>
</head>
<body>
	<nav class="navbar is-link">
		<div class="navbar-brand">
			<a class="navbar-item" href="javascript:void(0)">
				ES 654 Machine Learning
			</a>
		</div>
	</nav>
	<div class="container" id="app" style="margin-top: 40px">
		<div class="box">
			<div class="columns">
				<section class="hero">
					<div class="hero-body">
						<div class="container">
							<h1 class="title">Block Co-ordinate Descent</h1>
							<h2 class="subtitle">Neelay Upadhyaya, Chinmay Sonar, Vivek Srivastava, Sreejith Srikrishnan</h2>
						</div>
					</div>
				</section>
			</div>
			<div class="content">
				<div class="columns">
					<div class="column is-6">
						<div class="field is-horizontal">
							<div class="field-label is-normal">
								<label class="label">k-value</label>
							</div>
							<div class="field-body">
								<div class="field">
									<div class="control">
										<input id="k_value" class="input" type="number" placeholder="k-value" value="1" min="1" max="10">
									</div>
								</div>
							</div>
						</div>
						<div class="field is-horizontal">
							<div class="field-label is-normal">
								<label class="label">Number of rows:</label>
							</div>
							<div class="field-body">
								<div class="field">
									<div class="control">
										<input id="rows" class="input" type="number" min="500" max="2000" value="700" step="50">
									</div>
								</div>
							</div>
						</div>
						<div class="field is-horizontal">
							<div class="field-label is-normal">
								<label class="label">Number of columns:</label>
							</div>
							<div class="field-body">
								<div class="field">
									<div class="control">
										<input id="columns" class="input" type="number" min="1" max="15" value="8">
									</div>
								</div>
							</div>
						</div>
						<div class="field is-horizontal">
							<div class="field-label is-normal">
								<label class="label">Number of iterations:</label>
							</div>
							<div class="field-body">
								<div class="field">
									<div class="control">
										<input id="iterations" class="input" type="number" min="200" max="4000" value="600" step="100">
									</div>
								</div>
							</div>
						</div>
					</div>
					<div class="column">
						<div class="field">
							<label class="label">Strategy</label>
							<div class="control is-expanded">
								<div class="select is-multiple is-fullwidth">
									<select multiple id="strategy">
										<option value="greedy" selected="selected">Greedy</option>
										<option value="optimal">Optimal</option>
										<option value="fixed">Fixed</option>
										<option value="random">Random</option>
									</select>
								</div>
							</div>
						</div>
						
						<div class="field">
							<p class="control">
								<a class="button is-link" id="submit">Submit</a>
								<a class="button is-danger" id="clear">Clear</a>
							</p>
						</div>
					</div>
				</div>
			</div>
		</div>
		<div class="card" id="results" hidden="hidden">
			<progress class="progress is-small is-link" max="100">35%</progress>
			<div class="card-content" style="min-height: 250px">
				<div id="error-chart-containter"></div>
				<div id="time-chart-containter"></div>
			</div>
		</div>
	</div>
	<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
	<script src="https://code.highcharts.com/highcharts.src.js"></script>
	<script>
		/**
		 * (c) 2010-2019 Torstein Honsi
		 *
		 * License: www.highcharts.com/license
		 *
		 * Grid-light theme for Highcharts JS
		 * @author Torstein Honsi
		 */
		/* global document */
		// Load the fonts
		Highcharts.createElement('link', {
		    href: 'https://fonts.googleapis.com/css?family=Dosis:400,600',
		    rel: 'stylesheet',
		    type: 'text/css'
		}, null, document.getElementsByTagName('head')[0]);

		Highcharts.theme = {
		    colors: ['#7cb5ec', '#f7a35c', '#90ee7e', '#7798BF', '#aaeeee', '#ff0066',
		        '#eeaaee', '#55BF3B', '#DF5353', '#7798BF', '#aaeeee'],
		    chart: {
		        backgroundColor: null,
		        style: {
		            fontFamily: 'Dosis, sans-serif'
		        }
		    },
		    title: {
		        style: {
		            fontSize: '16px',
		            fontWeight: 'bold',
		            textTransform: 'uppercase'
		        }
		    },
		    tooltip: {
		        borderWidth: 0,
		        backgroundColor: 'rgba(219,219,216,0.8)',
		        shadow: false
		    },
		    legend: {
		        itemStyle: {
		            fontWeight: 'bold',
		            fontSize: '13px'
		        }
		    },
		    xAxis: {
		        gridLineWidth: 1,
		        labels: {
		            style: {
		                fontSize: '12px'
		            }
		        }
		    },
		    yAxis: {
		        minorTickInterval: 'auto',
		        title: {
		            style: {
		                textTransform: 'uppercase'
		            }
		        },
		        labels: {
		            style: {
		                fontSize: '12px'
		            }
		        }
		    },
		    plotOptions: {
		        candlestick: {
		            lineColor: '#404048'
		        }
		    },


		    // General
		    background2: '#F0F0EA'

		};

		// Apply the theme
		Highcharts.setOptions(Highcharts.theme);
	</script>
	<script>
		$(function() {
			var error_chart = undefined;
			var time_chart = undefined;
			$('#columns').on('input', function() {
				console.log($(this).val());
				$('#k_value').attr('max', $(this).val());
			});
			var destroyMaps = function() {
				if(error_chart) {
					error_chart.destroy();
				}
				if(time_chart) {
					time_chart.destroy();
				}
			}
			$('#clear').on('click', function(e) {
				e.preventDefault();
				destroyMaps();
				$('#results').hide();
				$('.progress').hide();
			});

			$('#submit').on('click', function(e) {
				e.preventDefault();
				$('#results').show();
				$('.progress').show();
				destroyMaps();
				var strategy = $('#strategy').val().join(',');
				var k_value = $('#k_value').val();
				var rows = $('#rows').val();
				var columns = $('#columns').val();
				var iterations = $('#iterations').val();
				console.log(strategy);
				var params = {
					strategy: strategy, 
					k_value: k_value, 
					rows: rows, 
					columns: columns, 
					iterations: iterations
				};
				console.log(params);
				var that = this;
				$(this).addClass('is-loading');
				$.get('http://10.1.139.96:5000/get/1/results',
					params,
					function(response, status) {
						$(that).removeClass('is-loading');
						$('.progress').hide();
						error_chart = Highcharts.chart('error-chart-containter', response['error_chart']);
						if($('#strategy').val().length > 1) {
							time_chart = Highcharts.chart('time-chart-containter', response['time_chart']);
						}
				}, 'json');
			});
		});
	</script>
</body>
</html>

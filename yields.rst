.. _yields:

******
Plotting yield to maturity of Government STRIPS
******
========
What are STRIPS?
========
STRIPS stands for "Treasury's Separate Trading of Registered Interest and Principal Securities." They are fixed income products created by banks and backed by United States government debt. While the United States Treasury does not create zero-coupon debt with a maturity over ninety days, investment banks synthetically create STRIPS by removing the interest payments from a regular treasury note or bond.

They are structured such that an investment bank will create zero coupon bonds based on the interest payment and principal of normal treasury notes. The bank sells the claim on each payment separately, rather than the Treasury note in its entirety.

========
Scraping the data
========

We scraped data off `the Wall Street Journal's`_ website and calculated the yield to maturity and number of days to maturity. We then plotted each data point in plotly.

Our first iteration of the graph is below. Every time you run the code, the data downloads from the website and the graph is re-plot. Our problem now was to get the data to remain on the graph over multiple days, in order to build a surface plot over time.

============
Version One
============

=========
The graph
=========
.. raw:: html

  <div style="margin-top:10px;">
    <iframe width="900" height="800" frameborder="0" scrolling="no" src="https://plot.ly/~cknovy/2.embed"></iframe>
  </div>
========
The code
========
The code for this graph is: ::

  #Importing required modules
  import requests
  from bs4 import BeautifulSoup
  import pandas as pd
  from datetime import *
  import numpy as np
  import plotly.plotly as py
  from plotly.graph_objs import *

  #Opening file containing data.
  inFile = open("data2.csv")
  dataFile = pd.read_csv(inFile, index_col=0, parse_dates="True")
  print "Data has been read."

  #Parsing HTML
  url = "http://www.wsj.com/mdc/public/page/2_3020-tstrips.html"
  r = requests.get(url)
  soup = BeautifulSoup(r.text, "lxml")

  #Creating empty variables to scrape data into.
  maturity = []
  bidPrice = []
  dtm = []
  ytm = []
  exp = []

  #Creating an object of the first object that is a dataframe.
  table = soup.find(class_='mdcTable')
  #Finding all the <tr> tag pairs, less the first few.
  for row in table.find_all('tr')[75:207]:
      #Create a variable of all the <td> tag pairs within each <tr> pair
      col = row.find_all('td')
      #Create variable of string inside first column.
      col1 = col[0].string.strip()
      #and then add to the appropriate variable.
      maturity.append(col1)
      #create var of float inside second column.
      col2 = col[1].string.strip()
      bidPrice.append(col2)


  ##Formatting dates and calculating days to maturity
  start = datetime.today().date()

  for item in maturity:
      date = datetime.strptime(item, "%Y %b %d").date()
      days = date - start
      dtm.append(days)
      ytm.append([])

  ##Creating the data frame.

  #Makes todays date into a string so that we can differentiate columns as they are appended to the data set.
  today = str(start)

  #Naming columns for dataframe
  columns = {'maturity': maturity, 'bidPrice': bidPrice, 'dtm': dtm, 'ytm': ytm, 'day': today}

  df = pd.DataFrame(columns)

  #Changing datatypes
  df['bidPrice'] = pd.to_numeric(df['bidPrice'], errors="ignore")
  df['ytm'] = pd.to_numeric(df['ytm'], errors="ignore")
  df['dtm'] = df['dtm'] / np.timedelta64(1, 'D')

  ##Calculating yield
  df['ytm'] = (100.0/df['bidPrice'])**((1/df['dtm'])*365)

  ##Adding today's data to master file.
  frames = [dataFile, df]
  #result.append(df, ignore_index=True)
  #print result

  ##Writing new dataframe to csv file. This write the ENTIRE dataset over again so don't mess it up.
  df.to_csv('data2.csv', mode='w')
  print "Data file updated."

  #Making the graph --  right now this only does the most current day.
  trace1 = Scatter(
      x = df['dtm'],
      y = df['ytm'],
      name = 'Yield',
      marker = Marker(color='rgb(234, 153, 153)')
  )
  layout = Layout(
      title = 'Price of STRIPS',
      xaxis = XAxis(
          showgrid = False,
          ),
      yaxis = YAxis(
          title='Yield',
          showline=False
          ),
          barmode='group'
  )
  data = Data([trace1])
  fig = Figure(data=data, layout=layout)
  plot_url = py.plot(fig, filename = "Govt STRIPS")


.. _the Wall Street Journal's: http://www.wsj.com/mdc/public/page/2_3020-tstrips.html

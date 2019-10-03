
HDBpp Adding Attributes
Skip to end of metadata
Created by Manuel Broseta Sebastià, last modified by Sergio Rubio Manrique on Feb 13, 2019 Go to start of metadata
Adding an attribute if you don't know which archiver nor database
Beamlines
Service Area
Add attributes to an specific database/archiver
Knowing which archiver corresponds to each attribute
Adding attributes that push event by code:
Adding attributes that need polling
Loading the attributes from a previous JSON file
Check that attributes are properly archived
Example from CLIC BPMs Data (service area)
Adding an attribute if you don't know which archiver nor database
Beamlines
You must choose the database (usually "hdbpp") and then the eventsubscriber (choose one from existing ones or create a ticket if all of them have already >1000 attributes assigned).

Service Area
In service area multiple databases exist, so you must use the PyTangoArchiving.multi module to select the proper database for each attribute.

This process is done using the "AttributeFilter" property of each EventSubscriber device; that contains a set of regexps to be used to assign attributes to subscribers.

import PyTangoArchiving as pta, fandango as fn
 
 
attrs = rd.get_attributes()
attrs = fn.find_attributes('lab/vc/elotech-01/temperature_[0-9]')
 
 
# This step assumes that archiving events were already configured in Jive
attrs = pta.multi.start_archiving_for_attributes(attrs)

Add attributes to an specific database/archiver
hdbpp = pta.api('hdbpp') 
fn.tango.set_attribute_events('sr/ct/gateway/current',polling=1000,arch_abs_event=0.01,abs_event=0.01) 
hdbpp.start_archiving('sr/ct/gateway/current','archiving/es/hdbpp-01',start=True)

hdbpp.load_last_values('sr/ct/gateway/current')

{'sr/ct/gateway/current': [1531493061.353, 
  150.73473083496157, 
  0, 
  '2018-07-13 16:44:21']}
Knowing which archiver corresponds to each attribute
import fandango as fn, PyTangoArchiving as pta
pta.multi.get_archivers_for_attributes('sr/ct/gateway/current') 
('tango://alba03.cells.es:10000/archiving/es/hdbpp-01', 1, ['ct']) 
Out[40]: {'tango://alba03.cells.es:10000/archiving/es/hdbpp-01': ['sr/ct/gateway/Current']}
Adding attributes that push event by code:
TO BE COMPLEMENTED BY MANOLO

Remember to add these two calls in your code: set_archive_event(attribute, True, False) and push_archive_event(attribute ,... )

import PyTangoArchiving.hdbpp as ptah, PyTangoArchiving as pta, fandango as fn 

lyrtech = fn.find_attributes('tango://alba02:10000/wr/rf/*lyrtechfacadeloops-*/*') 
hdbpp = ptah.HDBpp('hdbrf','localhost','manager','manager') 
hdbpp.add_attributes(lyrtech[:200],'tango/hdbpp-es/01-01',code_event=True,rel_event=-1,per_event=-1,abs_event=-1,start=True) 
hdbpp.add_attributes(lyrtech[200:],'tango/hdbpp-es/01-02',code_event=True,rel_event=-1,per_event=-1,abs_event=-1,start=True) 
Adding attributes that need polling
These call will setup the polling period and the event config.
If attributes were already polled, just omit the period argument.
thomson = fn.find_attributes('tango://alba02:10000/wr/rf/*thomson*/*') 
hdbpp.add_attributes(thomson,'tango/hdbpp-es/01-03',abs_event=1,rel_event=0.5,period=3000,start=True) 
Loading the attributes from a previous JSON file
allattrs = json.load(open('/homelocal/sicilia/allattrs.json'))

attrs1 = [l.split(';')[0].replace('alba02','tlarf01') for l in allattrs['tango/hdbpp-es/01-01']['Attribut
eList']] 

attrs2 = [l.split(';')[0].replace('alba02','tlarf01') for l in allattrs['tango/hdbpp-es/01-02']['Attribut
eList']] 
attrs3 = [l.split(';')[0].replace('alba02','tlarf01') for l in allattrs['tango/hdbpp-es/01-03']['AttributeList']]

hdbpp = pta.Reader().configs['hdbpp']
pta.api('hdbpp')

arch = fn.tango.get_full_name('tango/hdbpp-es/01-01')

for a in attrs1:

   hdbpp.start_archiving(a,arch,code_event=True)

arch = fn.tango.get_full_name('tango/hdbpp-es/01-02')

for a in attrs2:

   hdbpp.start_archiving(a,arch,code_event=True)



arch = fn.tango.get_full_name('tango/hdbpp-es/01-03')



for a in attrs3:



   hdbpp.start_archiving(a,arch, abs_event=1,rel_event=0.5,period=3000)



In [20]: hdbpp.load_last_values(attrs3[0])
Out[20]:  
{u'tango://tlarf01.cells.es:10000/wr/rf/thomson-plc-01/coolingdelay': [1523543926.247, 
 0L, 
 0, 
 '2018-04-12 16:38:46']}


Check that attributes are properly archived
Check that attributes have been properly archived:

data = {} 
for a in lyrtech: 
    try: 
        last = hdbpp.load_last_values(a) 
        data[a] = last 
    except: 
        data[a] = None 

failed = len([k for k,v in data.items() if v is None]) 
 

Example from CLIC BPMs Data (service area)
CS-14871 - SHUTDOWN: SR07 BPMs, problems with archiving events (clic) RESOLVED
 

import fandango as fn, PyTangoArchiving
rd = PyTangoArchiving.reader.Reader('*')
hdbpp = rd.configs['hdbpp']
attrs = """sr07/di/bpm-09/SaX
sr07/di/bpm-09/SaY
sr07/di/bpm-10/SaX
sr07/di/bpm-10/SaY""".split()

hdbpp.start_archiving(attrs[0],archiver='archiving/es/hdbpp-di-01',period=3000,rel_event=1,per_event=60000)
hdbpp.start_archiving(attrs[1],archiver='archiving/es/hdbpp-di-01',period=3000,abs_event=1,per_event=60000)
hdbpp.start_archiving(attrs[2],archiver='archiving/es/hdbpp-di-01',period=3000,abs_event=1,per_event=60000)
hdbpp.start_archiving(attrs[3],archiver='archiving/es/hdbpp-di-01',period=3000,abs_event=1,per_event=60000)

hdbpp.load_last_values(attrs,n=3)

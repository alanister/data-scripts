

import os
import re
from subprocess import call

__author__ = "Ramiro Alanis"
__maintainer__ = "Ramiro Alanis"
__email__ = "ramiro.alanis@gmail.com"
__status__ = "Development in 2.7.10"


def parse_report(r, d):
	"""Parses a Ringtail generated report for relevant metadata to reconstruct the native file name and directory path in the Ringtail native repository.
		
	Returns: 
		2 dictionaries:
		natives - {'DocID.ext':'UNC_PATH\DocID.ext'}
		doc_ids - {'DocID':'UNC_PATH\DocID.ext'}
	"""
	
	natives = {}
	doc_ids = {}
	for line in r:
		if line[0].isdigit():
			words = line.split(',')
			filename = native_id(words[1],words[2],words[4])
			doc_ids[words[1]] = natives[filename] = os.path.join(d, words[3].replace('/', '\\').rstrip('\n'))	#Concatenate the case root directory and the level structure
		else: continue
	return (natives, doc_ids)

	
def native_id(doc_id,doc_ext,doc_type):								#Function reconstructs the native file name based on report data
	if doc_type.rstrip('\n') == 'Email':
		return str(doc_id + '.msg')
	else:
		return doc_id + '.' + doc_ext

		
def copy_files(s, d, fname):
	call([r"robocopy", s, d, fname, r'/e'])

	
def yes_or_no(question):
	reply = str(raw_input(question+' (y/n): ')).lower().strip()
	if reply[0] == 'y': return True
	if reply[0] == 'n': return False
	else:
		return yes_or_no(question) 

		
def main():		
	rtdata = 'PATH goes here'								#Working directories	
	analyze_dir = 'PATH goes here'


	case = str(raw_input('enter the case name: ')).strip()				#Test if case path exists
	source = os.path.join(rtdata, case)									#Case path parameter to be passed to function: parse_report()
	if os.path.exists(source) == False:
		print source + ' unable to locate case, try again.'
		exit() 


	fname = str(raw_input('enter ringtail report name (csv format): ')).strip()		#Test if given report exists in current directory
	try:
		report = open(fname)
	except:
		print 'report file cannot be opened:', fname
		exit()

		
	(files, docids) = parse_report(report, source)						#Create dictionaries
	report.close()

	for k, v in files.items():											#v = source, k = native file
		copy_files(v, neevia_in, k)										#robocopy one file at a time

		
if __name__ == '__main__':
    main()
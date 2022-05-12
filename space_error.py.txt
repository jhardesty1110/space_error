#! /usr/bin/env python

"""
  Author:  Jim Hardesty

  This is a script that takes a caloutput.db file and error code as the inputs, and
  looks for the worst case spacing errors.

"""


import sys, getopt
import os
import types
import csv
import re
import subprocess


def usage():
    print('''
    usage:

                      **** For edge to edge spacing rule only ****
		      
	The error has to be of the form:
	
    GRDS226
    4904 4904 2 Aug  9 10:56:46 2019      
    Rule File Pathname: runcal.cal
    PCFILL minimum space to BI >= 1.50 um.
    e 1 2
    135094 13886 135094 13954 
    135220 13805 135220 14010 
    e 2 2
    135094 33986 135094 34054 
    135220 33905 135220 34135 
    e 3 2
    135094 60386 135094 60454 
    135220 60305 135220 60500
    
		      
		      
		      
		      
    
    space_error.py -p "proj dir which contains the .db file" -r "the rule to look for"

    ex:    -r GRDS226

    or don't include  -p "proj dir which contains the .db file"   and be in the directory with
    the caloutput.db file

    ''')


def main(argv):
    opt_p = ''     # project directory
    opt_r = ''     # the spacing rule under analysis

    try:
         opts, args = getopt.getopt(argv,"hp:r:", ["proj_dir=","spacing_rule="])
    except getopt.GetoptError:
         print 'Something went wrong, maybe wrong option letter'
	 print 'space_error.py -p "proj dir which contains the .db file" -r "the rule to look for"'
	 sys.exit()

    for opt, arg in opts:

         if opt == '-h':
	      usage()
	      sys.exit()
         elif opt in ('-p', '--proj_dir'):
	      opt_p = arg
	 elif opt in ('-r', '--spacing_rule'):
	      opt_r = arg

    if opt_p == '':
         opt_p = os.getcwd()


    if opt_r == '':
         print '\nPlease put in the desired spacing rule.'
	 usage()
         sys.exit()
 


    #  If the .db file is not called caloutput.db, this code will need to be changed.
    #  Also, if the spacing violation is not of the following form, this code will need to be changed.
    #
    #
    #
    # GRDS226
    # 4904 4904 2 Aug  9 10:56:46 2019      
    # Rule File Pathname: runcal.cal
    # PCFILL minimum space to BI >= 1.50 um.
    # e 1 2
    # 135094 13886 135094 13954 
    # 135220 13805 135220 14010 
    # e 2 2
    # 135094 33986 135094 34054 
    # 135220 33905 135220 34135 
    # e 3 2
    # 135094 60386 135094 60454 
    # 135220 60305 135220 60500
    #



    db_file_list = open(opt_p + "/caloutput.db", "r")
    #db_file_list = open(opt_p + "/mk5767e_01.db", "r")

    db_file_list_contents = [line.rstrip() for line in db_file_list]       #convert the text file into a list.  Takes out the newline character and trailing whitespace.

    violation_list = []
    
    item_iter = iter(db_file_list_contents)    #make db_file_list_contents iterable


    #print db_file_list_contents




    found_it_flag = "no"

    for line in item_iter:


         #print line



         if found_it_flag == "yes":
	      if re.search("GR", line):
	           found_it_flag = "no"
		   item_iter.next()
              else:
	           line2 = line.rstrip()   #remove trailing spaces
		   violation_list.append(line2)
		   #print line2

         if found_it_flag == "no":
              #if opt_r.upper() in line:
              if opt_r in line:
	           found_it_flag = "yes"
		   violation_list.append(line)
		   #print line


    if not len(violation_list):
         print "\nCould not find the violation rule " + opt_r + " in caloutput.db\n"
         sys.exit()

    #for item in violation_list:
         #print item


    violation_list_iter = iter(violation_list)

    space_violation_list = []

    for item in violation_list_iter:
         tmp_list1 = []
	 tmp_list2 = []
         if re.search(r'^e ', item):

              tmp_list1 = list(violation_list_iter.next().split())
              #print tmp_list1


	      tmp_list2 = list(violation_list_iter.next().split())
              #print tmp_list2

	      if float(tmp_list1[2]) - float(tmp_list1[0]) == 0:
	           if float(tmp_list2[2]) - float(tmp_list2[0]) == 0:
		        space_violation = abs(float(tmp_list2[2]) - float(tmp_list1[2]))
			space_violation_list.append(space_violation/100)

	      if float(tmp_list1[3]) - float(tmp_list1[1]) == 0:
	           if float(tmp_list2[3]) - float(tmp_list2[1]) == 0:
		        space_violation = abs(float(tmp_list2[3]) - float(tmp_list1[3]))
                        space_violation_list.append(space_violation/100)

    if not len(space_violation_list):
         print "\nCould not find any spacing violation data under the rule " + opt_r + "\n"
	 sys.exit()

    #for item in space_violation_list:
         #print str(item) + " um"

    print "\nlength of spacing violation list = " + str(len(space_violation_list)) + "\n"

    # find min and max values in spacing list

    min_value = space_violation_list[0]
    max_value = space_violation_list[0]

    for item in space_violation_list:

         if item < min_value:
	      min_value = item

	 if item > max_value:
	      max_value = item


    #print "item 0 value = " + str(space_violation_list[0]) + " um"

    print "minumum spacing value = " + str(min_value) + " um"
    print "maximum spacing value = " + str(max_value) + " um\n"

    print "spacing rule = " + opt_r + "\n"
    #print "opt_p = " + opt_p + "\n"




    db_file_list.close()

if __name__ == '__main__':

    main(sys.argv[1:])


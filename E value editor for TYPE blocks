# Python script to independently change E values for any ;TYPE: blocks
# in a Cura generated .gcode file using a multiplier

import time

# 0 = OFF, 1 = ON
debug = 0

# input file - change accordingly
ifile = open('type block.gcode', 'r')

# output file - change accordingly
ofile = open('type block_edited.gcode', 'w')

# blocks to alter with associated multiplier
# Usage: blocks = [("BLOCK1", multiplier1),("BLOCK2", multiplier2),("BLOCK3", multiplier3)]
blocks = [(";TYPE:FILL", 100.0 / 110.0)]

# number of decimal places of E value to process
precision = 5	# default 5

# do not change
precisionMult = pow(10, precision)



# function declarations
# convert E value from string to long
def EtoL(str):
	return long(str.split(".")[0]) * precisionMult + long(str.split(".")[1])

# convert E value from long to string
def LtoE(l):
	whole = int(l / precisionMult)
	fractional = long((l - whole * precisionMult))
	return str(whole) + "." + str(fractional).zfill(precision)



# add file header
ofile.write("\n")
ofile.write(";[---------------------------\n")
ofile.write("; E values altered by python script written by Steve Bollerud\n")
ofile.write(";    Processed:                %s %s %s %s at %s\n" % (time.strftime("%a"), time.strftime("%d"), time.strftime("%b"), time.strftime("%Y"), time.strftime("%X")))
ofile.write(";    Decimal places processed: %d\n" % precision)
ofile.write(";    Block(s) processed:       %s\n" % blocks)
ofile.write("\n")



startE = 0L
currentE = 0L
newE = 0L
multiplier = 0
retract = 0
State = 0  # 0 - Pre, 1 - Main, 2 - Block, 3 - Post



# BEGIN main processing
# read first line from input file
line = ifile.readline()
while line != '':
	if State == 0:  #State.Pre
		# scan for first layer
		if line.count(';LAYER:0') == 1:
			State = 1  #State.Main

		if debug == 1:
			print("1 " + line)

		ofile.write(line)
		line = ifile.readline()

	elif State == 1:  #State.Main:
		if line.count(';End GCode') == 1:

			if debug == 1:
				print("2 " + line)

			ofile.write(line)
			line = ifile.readline()
			State = 3  #State.Post

		elif line.count(";TYPE:") == 1:
			for b, m in blocks:
				if line.count(b) == 1:
					State = 2  #State.Block
					multiplier = m
					startE = currentE
		
			if debug == 1:
				print("3 " + line)
	
			ofile.write(line)
			line = ifile.readline()

		else:
			if line.count(" E") == 1:
				# save for calculating delta
				previousE = currentE
				currentE = EtoL(line.split("E")[1])
				deltaE = currentE - previousE

				# if delta < 0 then retraction must be on - maintain actual retraction value
				if deltaE < 0:
					retract = 1
				elif retract == 1:
					retract = 0

				newE = newE + deltaE

				if debug == 1:
					print("4 " + line.split("E")[0] + "E" + LtoE(newE) + "\n")

				# write altered line containing E value to output file
				ofile.write(line.split("E")[0] + "E" + LtoE(newE) + "\n")
			else:

				if debug == 1:
					print("5 " + line)

				ofile.write(line)

			line = ifile.readline()

	elif State == 2:  #State.Block:
		if line.count(';End GCode') == 1:

			if debug == 1:
				print("6 " + line)

			ofile.write(line)
			line = ifile.readline()
			State = 3  #State.Post
		
		elif line.count(";TYPE:") == 1:
			for b, m in blocks:
				if line.count(b) == 1:
					multiplier = m
					startE = newE
			else:
				State = 1  #State.Main

			if debug == 1:
				print("7 " + line)

			ofile.write(line)
			line = ifile.readline()

		else:
			if line.count(" E") == 1:
				previousE = currentE
				currentE = EtoL(line.split("E")[1])
				deltaE = currentE - previousE

				# if delta < 0 then retraction must be on - maintain actual retraction value
				if deltaE < 0:
					retract = 1
				else:
					if retract == 1:
						retract = 0
					else:
						# must be normal extrusion - use multiplier
						deltaE = deltaE * multiplier

				newE = newE + deltaE

				if debug == 1:
					print("8 " + line.split("E")[0] + "E" + LtoE(newE) + "\n")

				# write altered line containing E value to output file
				ofile.write(line.split("E")[0] + "E" + LtoE(newE) + "\n")
			else:

				if debug == 1:
					print("9 " + line)

				ofile.write(line)

			line = ifile.readline()

	elif State == 3:  #State.Post:

		if debug == 1:
			print("0 " + line)

		ofile.write(line)	
		line = ifile.readline()

ifile.close()
ofile.close()

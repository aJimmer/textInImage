from PIL import Image
import binascii
import optparse
import os

def check_format(im):
	if(im.format == 'JPEG' and im.mode == 'RGB'):
		return True
	else:
		return False

def str_to_bin(msg):
	binary = bin(int(binascii.hexlify(msg), 16))
	return binary[2:]

def dec_to_bin(num):
	return '{0:09b}'.format(num)

def binary_to_str(binary):
	message = binascii.unhexlify('%x' % (int('0b' + binary, 2)))
	return message

# used to pad bit strings by appending and right circular shifting while the length is not divisible by 3
def rotate(strg, flag):

	count = 0

	while ((len(strg)) % 3) > 0: 
		strg += '0'
		count-=1	

	return strg[count:] + strg[:count]

# encode takes a pixel and a series of 0s/1s which will hide_input our data in the least significant
def encode(pixelSection, bNum1, bNum2, bNum3):

	r = list(dec_to_bin(pixelSection[0]))
	g = list(dec_to_bin(pixelSection[1]))
	b = list(dec_to_bin(pixelSection[2]))

	# print( 'RGB after binary: ' + str(r) + ',' + str(g) + ',' + str(b))
	
	# convert R
	if(r[-1] != bNum1):
		r[-1] = str(bNum1)
	r = int(''.join(r), 2)

	# convert G
	if(g[-1] != bNum2):
		g[-1] = str(bNum2)
	g = int(''.join(g), 2)

	# convert B
	if(b[-1] != bNum3):
		b[-1] = str(bNum3)
	b = int(''.join(b), 2)

	# print( 'RGB after decimal: ' + str(r) + ',' + str(g) + ',' + str(b))
	return tuple((r, g, b))

# decode takes a pixel and extracts the least significant bits and returns a bit string with our decoded values
def decode(pixel):

	rList = list(dec_to_bin(pixel[0]))
	gList = list(dec_to_bin(pixel[1]))
	bList = list(dec_to_bin(pixel[2]))

	lsRed = rList[-1]
	lsGreen = gList[-1]
	lsBlue = bList[-1]

	# print(rList,gList,bList)
	# print(lsRed,lsGreen,lsBlue)
	return lsRed + lsGreen + lsBlue

# hide_input takes the image file to encode along with a message to embed into it
def hide_input(filename, msg):

	og_msg = msg
	bin_msg = '' # message in binary form
	image_in = None 
	image_out = None
	reversed_new_data = []
	new_data = [] # tuble will be used to build new image
	msg_length = 0 # integer value for the length of our message
	bin_msg_length = 0 # binary value for the length of our message
	msg_counter = 0 # index to the binary message 
	data = None # used to extract data from the existing image

	# extract the file name and extension
	name, ext = os.path.splitext(filename)

	if msg == 'source.txt':
		file = open(msg,'r')
		og_msg = file.read()

	# Open the file provided by the filename and cconvert to JPEG/RGB if necessary
	try:
		image_in = Image.open(filename)
		image_out = image_in

		pixel = image_in.load()

		if check_format(image_in) == False :
			print('format is incorrect!')	

			# Format is incorrect so convert to a JPEG/RGB image)	
			image_out = Image.new('RGB', image_in.size, (255, 255, 255))
			image_out.paste(image_in)
			image_out.save(name + '.jpg', 'JPEG', quality=80)

			# Attempt to open converted image
			try:
				image_in = Image.open(name + '.jpg')
			except:
				print('failed reopen modified')
				return
		else:
			print('Opened file successfully!') 
	except:
		print('Failed to open the image!')
		return

	# convert message to binary and append a delimiter 
	# print('message to encode: ' + og_msg)

	bin_msg = rotate(str_to_bin(og_msg),'str')
	# print('binary msg: ' + bin_msg)
	msg_length = len(bin_msg)
	data = image_in.getdata()
	
	bin_msg_length = rotate(dec_to_bin(msg_length),'int')
	# print(str(msg_length) + ' in binary is: ' + bin_msg_length)
	# print(bin_msg)
	bin_index = 0
	first_eleven_pixels = 11

	#iterate and reorder the data to manipulate from bottom right pixel to top left
	for pixel in reversed(data):

		# add msg length to pixels
	 	if bin_index > (0 -len(bin_msg_length)):
	 		first_eleven_pixels -= 1
	 		new_data.append(encode(pixel, bin_msg_length[bin_index-3], bin_msg_length[bin_index - 2], bin_msg_length[bin_index - 1]))
	 		bin_index -= 3

	 	# fill the remaining of 11 pixels with 0's
		elif first_eleven_pixels > 0:
			first_eleven_pixels -= 1
			new_data.append(encode(pixel, 0, 0, 0))

		# add msg to pixels
		elif first_eleven_pixels == 0 and msg_counter > (0 - msg_length):
			new_data.append(encode(pixel, bin_msg[msg_counter - 3], bin_msg[msg_counter - 2], bin_msg[msg_counter - 1]))
			msg_counter -= 3

		#keep remaining pixels
		else:
			new_data.append(pixel)
	
	# reorder the data to print the image from top left pixel to bottom right
	for data in reversed(new_data):
		reversed_new_data.append(data)

	# save image as a PNG
	image_out.putdata(reversed_new_data)
	image_out.save(name + '.png', 'PNG')

	return 'Successfully encoded!'

def retrieve_output(filename):

	image_in = None
	data = None
	first_eleven_pixels = 11
	msg_length = 0
	decoded_msg = ''
	bin_msg = ''
	bin_msg_length = ''

	# attempt to open the image to be decode
	try:
		image_in = Image.open(filename)
		if image_in.format != 'PNG' :
			print('format is incorrect!')	
			return
		else:
			print('format is correct!')

	except:
		print('failed to open the image')
		return

	data = image_in.getdata()

	# reorder the data to extract from bottom right to top left
	for pixel in reversed(data):
		# extract the length of the message stored in the first eleven pixels
		if first_eleven_pixels > 0:
			first_eleven_pixels -= 1
			bin_msg_length = decode(pixel) + bin_msg_length
			if first_eleven_pixels == 0:
				msg_length = int(bin_msg_length, 2)

		# iterate trough the pixels and extract 3 values at a time
		elif msg_length > 0:
			msg_length -= 3
			bin_msg = decode(pixel) + bin_msg 
			if msg_length == 0:
				decoded_msg = binary_to_str(bin_msg)

	print(decoded_msg)
	return ('Successfully decoded!')

def main():

	# options functionality 
	parser = optparse.OptionParser('usage %prog ' + '-e/-d <target text>' + '-i/-o <target file>')
	parser.add_option('-i', dest='hide_input', type='string', help='target picture path to hide_input text')
	parser.add_option('-o', dest='retrieve_output', type='string', help='target picture path to retrieve_outputieve text')
	parser.add_option('-e', dest ='input', type='string', help='text to encrypt')
	parser.add_option('-d', dest = 'output', type='string', help='image to decrypt')
	# parse arguments and 
	(options, args) = parser.parse_args()

	# encode and decode options
	if (options.hide_input != None and options.input != None):
		print(hide_input(options.hide_input, options.input))
	elif (options.retrieve_output != None and options.output != None):
		print(retrieve_output(options.retrieve_output))
	else:
			print(parser.usage)
			exit(0)

if __name__ == '__main__':
	main()

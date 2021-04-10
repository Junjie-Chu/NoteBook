# ApachePulsar_tasks
tasks of computer lab 1 in Data Engineering 2, Uppsala Universiy, 2021 spring
solutions: Junjie Chu
## task1
run pulsar, 1 producer and 1 consumer in the same VM  
producer.py:  
```python
#python2 producer.py
import pulsar
# Create a pulsar client by supplying ip address and port
client = pulsar.Client('pulsar://localhost:6650')
# Create a producer on the topic that consumer can subscribe to
producer = client.create_producer('DEtopic')
# Send a message to consumer
producer.send(('Welcome to Data Engineering Course!').encode('utf-8'))
# Destroy pulsar client
client.close()
```
consumer.py:  
```python
import pulsar
# Create a pulsar client by supplying ip address and port
client = pulsar.Client('pulsar://localhost:6650')
# Subscribe to a topic and subscription
consumer = client.subscribe('DEtopic', subscription_name='DE-sub')
# Display message received from producer
msg = consumer.receive()
try:
print("Received message : '%s'" % msg.data())
# Acknowledge for receiving the message
consumer.acknowledge(msg)
except:
consumer.negative_acknowledge(msg)# Destroy pulsar client
client.close()
```
## task2
run pulsar, 1 producer in the same VM   
run 1 consumer in another VM
My producer and pulsar are running on the same VM. So there is no need to change the content of producer.py.  
The VM I run pulsar on has a fixed ip address: 192.168.2.232 and a floating ip address:   
130.238.280.240. Thus, in consumer.py, I need to change the local host from localhost to 192.168.2.232.   
Floating ip is only to communicate with the outside world i.e., outside of the private network.    
Therefore, using fixed ip is the correct way to make communication.  
consumer.py:    
```python
import pulsar
# Create a pulsar client by supplying ip address and port
client = pulsar.Client('pulsar://192.168.2.232:6650')
# Subscribe to a topic and subscription
consumer = client.subscribe('DEtopic', subscription_name='DE-sub')
# Display message received from producer
msg = consumer.receive()

try:
  print("Received message : '%s'" % msg.data())
  
  # Acknowledge for receiving the message
  consumer.acknowledge(msg)
  
except:
  consumer.negative_acknowledge(msg)
  
# Destroy pulsar client
client.close()
```
## task3
diagram:  
![image](https://user-images.githubusercontent.com/65893273/114259608-bfe47f80-9a01-11eb-94be-dc7ec565131c.png)  
Steps：  
1. Run consumer4.py  
2. Run consumer1.py, consumer2.py, consumer3.py  
3. Run producer.py  
The process is as follows.  
1. producer 1 splits the sentence and sends word as well as the words’ sequential numbers to Topic 1.    
2. Consumer 1 ,2 and 3 get data from topic 1 with subscripition A in shared mode, do some operations.     
The consumer 1, 2 and 3 are also producer 1, 2, and 3. They also send capitalized words as well as their sequential numbers to topic 2.  
3. Consumer 4 gets data from topic 2 with subscripition B in exclusive mode, combine the words according to their order.  
the code of consumer 1,2,3 is the same.  
consumer4.py:  
```python
import pulsar
from conversion import *

# Create a pulsar client by supplying ip address and port
client = pulsar.Client('pulsar://localhost:6650')
# Subscribe to a topic and subscription
consumer = client.subscribe('task4-2', subscription_name='DE-subB')
# Display message received from producer
# msg = consumer.receive()

ITERATION = 11
resultant_string = ""
mylist = []
try:
  for i in range(0,ITERATION):
  	msg = consumer.receive()
  	print("Received message : '%s'" % msg.data())
  	# Acknowledge for receiving the message
  	consumer.acknowledge(msg)
  	
  	string = msg.data()
  	# Split string
	splited_string = string.split(" ")
	print(splited_string)
	mylist.append(splited_string)
  	#resultant_string +=   msg.data()  + ' '
  	
  print('Before sort:')
  print(mylist)
  mylist.sort(key=lambda x:int(x[1]))
  print('After sort:')
  print(mylist)
  
  for j in mylist:
  	resultant_string += j[0] + ' '		 
  print("Resultant String: '%s'" % resultant_string)
  
  
except:
  consumer.negative_acknowledge(msg)
  
# Destroy pulsar client
client.close()
```
consumer1.py
```python
import pulsar
from conversion import *

# Create a pulsar client by supplying ip address and port
client = pulsar.Client('pulsar://localhost:6650')
# Subscribe to a topic and subscription
consumer = client.subscribe('task4-1', subscription_name='DE-subA',consumer_type=pulsar.ConsumerType.Shared)
# Create a producer on the topic that consumer can subscribe to
producer = client.create_producer('task4-2') 
# Display message received from producer
# msg = consumer.receive()

ITERATION = 5
resultant_string = ""
while True:
  try:
  #for i in range(0,ITERATION):
  	msg = consumer.receive()
  	print("Received message : '%s'" % msg.data())
  	print(msg.message_id())
  	# Acknowledge for receiving the message
  	consumer.acknowledge(msg)
  	upper_case_string = conversion(msg.data(), function)
  	# Send a message to consumer
	producer.send((upper_case_string).encode('utf-8'))
	
  	#resultant_string +=   upper_case_string  + ' '
  	
  
  #print("Resultant String: '%s'" % resultant_string)
  
  
  except:
    consumer.negative_acknowledge(msg)


# Destroy pulsar client
client.close()
```
producer.py:  
```python
import pulsar
from conversion import *

# Create a pulsar client by supplying ip address and port
client = pulsar.Client('pulsar://localhost:6650')
# Create a producer on the topic that consumer can subscribe to
producer = client.create_producer('task4-1')

ITERATION = 11
# Input string
INPUT_STRING = "I want to be capatilized and want to be tested too"
# Split string
split_string = INPUT_STRING.split(" ")

for i in range(0,ITERATION):
	# Send a message to consumer
	producer.send((split_string[i]+' '+str(i)).encode('utf-8'))
	
# Destroy pulsar client
client.close()
```

conversion.py:  
```python
#!/usr/bin/env python3
import time

"""
    The implementation is used to demonstrate an intensive "conversion" function on elements
"""

# Fill in your author information
___author___ = ""
___email____ = ""

# Input string
INPUT_STRING = "I want to be capatilized"

# Iteration represents the operation applied to each word in the string 
# e.g., 1 represents that operation is applied to first word in the string
ITERATION = 5

def conversion(substring, operation):
    """A conversion function which takes a string as an input and outputs a converted string

    Args:
        substring (String)
        operation (function): This is an operation on the given input

    Returns:
        [String]: Converted String
    """


    # returns the conversion applied to input
    return function(substring)



def function(string):
    """ A function that performs some operation on a string. You can change the operation accordingly

    Args:
        string (String): input string on which some operation is applied

    Returns:
        [String]: string in upper case
    """
    return string.upper()





if __name__ == "__main__":

    # Check for correct input
    if ITERATION > len(INPUT_STRING.split()):
        print ("Iteration cannot be greater than the number of words in a string")
        print ("Terminating the benchmark")
        exit()

    print ("Original String: {}".format(INPUT_STRING))
    resultant_string = ""
    


    split_string = INPUT_STRING.split(" ")


    for i in range(0,ITERATION):

        upper_case_string = conversion(split_string[i], function)
        
        resultant_string +=   upper_case_string  + ' '    


    print ("Resultant String: {}".format(resultant_string))
```



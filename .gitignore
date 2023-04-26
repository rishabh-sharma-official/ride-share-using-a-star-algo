import networkx as nx
import time
import csv

inital_time = time.time()
last_van_update = time.time()
last_req_schedule = time.time()
stop_time = 900

#Creating a random graph with 100 nodes and connectivity 2
G = nx.gnp_random_graph(100, 0.2, 1000)

class Van:
  def __init__(self, vanId, currentLocation, customerCount):
    self.vanId = vanId
    self.vans_requestQueue = []
    self.vans_schedule_queue = {}
    self.distance_travelled = 0
    self.trips = 0
    self.van_route = []
    self.currentLocation = currentLocation
    self.customerCount = customerCount

class Request:
  name: str
  pickupNode: int
  dropoffNode: int
  def __init__(self, name, pickupPoint, dropoffPoint):
    self.name = name
    self.pickupPoint = pickupPoint
    self.dropoffPoint = dropoffPoint

#Returns shortest path length between 2 given nodes
def calculatePathLength(node1, node2):
  len = 0
  try:
    nx.dijkstra_path_length(G, node1, node2)
  except:
    print("Node migth not be connected")
  return len

#Returns A* path from node 1 to node 2
def getAStartPath(node1, node2):
  path = []
  try:
    path = nx.astar_path(G, node1, node2)
  except:
    print("No path is found")
  return path

#Initilizing global vans list and requests queue
vansQueue = []
requestsQueue = []

#vansQueue.append(Van(1, 0, 0))

#Creating 30 van objects
for i in range(1,61):
  vansQueue.append(Van(i, i, 0))

#Allocate a van to request based on min distance
def allocate_van(vansQueue, cust_location):
  min_dist = 10000
  alloc = 0
  id = 0
  for van in vansQueue:
    #print(van.vanId)
    if(len(van.vans_requestQueue) < 3):
      path_length = calculatePathLength(van.currentLocation, cust_location)
      if(path_length < min_dist):
        min_dist = path_length
        id = van.vanId
    else:
      if(len(van.vans_schedule_queue) < 4):
          ride_schedule(van)
  alloc = id
  print("Allocate van: ", alloc)

  for van in vansQueue:
    if(van.vanId == alloc):
      van.vans_requestQueue.append(requestsQueue.pop(0))

#Schedule the requests waiting in van's local request queue
def ride_schedule(vanObj):
  req_scheduling_queue = []
  print("Van Id: ", vanObj.vanId)

  #Based on space available in the van, requests are taken from local request queue for scheduling
  if(vanObj.customerCount == 2):
    req_scheduling_queue.append(vanObj.vans_requestQueue.pop(0))
  elif(vanObj.customerCount == 1):
    req_scheduling_queue.append(vanObj.vans_requestQueue.pop(0))
    req_scheduling_queue.append(vanObj.vans_requestQueue.pop(0))
  elif(vanObj.customerCount == 0):
    req_scheduling_queue.append(vanObj.vans_requestQueue.pop(0))
    req_scheduling_queue.append(vanObj.vans_requestQueue.pop(0))
    req_scheduling_queue.append(vanObj.vans_requestQueue.pop(0))
  
  print("Len of temp req queue: ", len(req_scheduling_queue))

  #scheduling with 1 customer
  if(len(req_scheduling_queue) == 1): 
    pickup = req_scheduling_queue[0].pickupPoint
    dropoff = req_scheduling_queue[0].dropoffPoint
    vanObj.vans_schedule_queue[pickup] = 'P'
    vanObj.vans_schedule_queue[dropoff] = 'D'
    req_scheduling_queue.clear() #clearing the temp scheduling queue after scheduling is completed
  #Scheduling with 2 customers
  elif(len(req_scheduling_queue) == 2):
    pickup1 = req_scheduling_queue[0].pickupPoint
    dropoff1 = req_scheduling_queue[0].dropoffPoint
    pickup2 = req_scheduling_queue[1].pickupPoint
    dropoff2 = req_scheduling_queue[1].dropoffPoint
    #Case1 - p1 is picked up first
    if(calculatePathLength(vanObj.currentLocation, pickup1) < calculatePathLength(vanObj.currentLocation, pickup2)):
      vanObj.vans_schedule_queue[pickup1] = 'P'
      #subcase - p1 is dropped off before p2 pickup
      if(calculatePathLength(pickup1, dropoff1) < calculatePathLength(pickup1,pickup2)):
        vanObj.vans_schedule_queue[dropoff1] = 'D'
        vanObj.vans_schedule_queue[pickup2] = 'P'
        vanObj.vans_schedule_queue[dropoff2] = 'D'
      else: #p2 is picked up before p1 drop off
        vanObj.vans_schedule_queue[pickup2] = 'P'
        #p1 is dropped off before p2
        if(calculatePathLength(dropoff1, pickup2) < calculatePathLength(dropoff2, pickup2)):
          vanObj.vans_schedule_queue[dropoff1] = 'D'
          vanObj.vans_schedule_queue[dropoff2] = 'D'
        #p2 is dropped off before p1
        else:
          vanObj.vans_schedule_queue[dropoff2] = 'D'
          vanObj.vans_schedule_queue[dropoff1] = 'D'
    #case2 - p2 is picked up first
    else:
      vanObj.vans_schedule_queue[pickup2] = 'P'
      #subcase- p2 is dropped off before picking p1 
      if(calculatePathLength(pickup2, dropoff2) < calculatePathLength(pickup2,pickup1)):
        vanObj.vans_schedule_queue[dropoff2] = 'D'
        vanObj.vans_schedule_queue[pickup1] = 'P'
        vanObj.vans_schedule_queue[dropoff1] = 'D'
      else: #p1 pick up is before p2 dropoff
        vanObj.vans_schedule_queue[pickup1] = 'P'
        #p1 is dropped off before p2
        if(calculatePathLength(pickup1, dropoff1) < calculatePathLength(pickup1, dropoff2)):
          vanObj.vans_schedule_queue[dropoff1] = 'D'
          vanObj.vans_schedule_queue[dropoff2] = 'D'
        #p2 is dropped off before p1
        else:
          vanObj.vans_schedule_queue[dropoff2] = 'D'
          vanObj.vans_schedule_queue[dropoff1] = 'D'
    req_scheduling_queue.clear()
  
  #Scheduling for 3 customers
  elif(len(req_scheduling_queue) == 3):
    pickup_list = {}
    dropoff_list = {}
    p1p = req_scheduling_queue[0].pickupPoint
    p1d = req_scheduling_queue[0].dropoffPoint
    p2p = req_scheduling_queue[1].pickupPoint
    p2d = req_scheduling_queue[1].dropoffPoint
    p3p = req_scheduling_queue[2].pickupPoint
    p3d = req_scheduling_queue[2].dropoffPoint

    pickup_list[p1p] = calculatePathLength(vanObj.currentLocation,p1p)
    pickup_list[p2p] = calculatePathLength(vanObj.currentLocation,p2p)
    pickup_list[p3p] = calculatePathLength(vanObj.currentLocation,p3p)
    sorted_pickups = sorted(pickup_list)
    #print(sorted_pickups)
    dropoff_list[p1d] = calculatePathLength(sorted_pickups[len(sorted_pickups)-1],p1d)
    dropoff_list[p2d] = calculatePathLength(sorted_pickups[len(sorted_pickups)-1],p2d)
    dropoff_list[p3d] = calculatePathLength(sorted_pickups[len(sorted_pickups)-1],p3d)
    sorted_dropoffs = sorted(dropoff_list)

    for pickup in sorted_pickups:
      vanObj.vans_schedule_queue[pickup] = 'P'
    
    for dropoff in sorted_dropoffs:
      vanObj.vans_schedule_queue[dropoff] = 'D'

    req_scheduling_queue.clear()
  
  print(vanObj.vans_schedule_queue)
  #Storing vans route
  completePath = getAStartPath(vanObj.currentLocation, list(vanObj.vans_schedule_queue.keys())[0])
  completePath.pop(0) #removing 1st node as it is same as current location
  vanObj.van_route = completePath

def updateVanProps():
  print("Inside update van")
  #Updating van's current location
  for van in vansQueue:
    if(len(van.van_route) != 0 ):
      van.currentLocation = van.van_route.pop(0)
      van.distance_travelled+= 1

    #If van has arrived pick up or drop off point, update van properties
    if van.vans_schedule_queue:
      if(van.currentLocation == list(van.vans_schedule_queue.keys())[0]):
        node_type = van.vans_schedule_queue.get(list(van.vans_schedule_queue.keys())[0])
        van.vans_schedule_queue.pop(list(van.vans_schedule_queue.keys())[0])
        completePath = getAStartPath(van.currentLocation, list(van.vans_schedule_queue.keys())[0])
        completePath.pop(0)
        van.van_route = completePath
        #Checking if arrived node is a drop off or pick up
        if(node_type == 'D'):
          print("Customer is dropped at ", van.currentLocation)
          van.trips+=1
          if van.vans_requestQueue:
            if(len(van.vans_schedule_queue) < 4):
              ride_schedule(van) #scheduling requests from local request queue as van will have space after dropoff
          van.customerCount-=1
        else:
          print("Customer is picked up at ", van.currentLocation)
          van.customerCount+=1

#Schedule the remaining requests in global request queue(runs every 4 mins)
def scheduleRequests():
  if requestsQueue:
    for req in requestsQueue:
      allocate_van(vansQueue, req.pickupPoint)

#Getting requests data from csv
with open("/content/requests_1000.csv", 'r') as file:
  csvreader = csv.reader(file)
  next(csvreader)
  for row in csvreader:
    cusmoterName = row[0]
    pickupNode = int(row[1])
    dropoffNode = int(row[2])
    #Checking if pick up node and drop off nodes are in range and not equal
    if(not((pickupNode < 0 or pickupNode > 100) or (dropoffNode < 0 or dropoffNode > 100)) and pickupNode != dropoffNode):
      requestsQueue.append(Request(cusmoterName, pickupNode, dropoffNode))
      allocate_van(vansQueue, pickupNode)
    else:
      print("Pick up node or drop off node is  either out of range or both are equal")
  
    if (time.time() > last_van_update + 120) : #scheduling updateVanProps func for every 2 mins
      last_van_update = time.time()
      updateVanProps()
    elif (time.time() > last_van_update + 240) : #scheduling scheduleRequests func for every 4 mins
      last_req_schedule = time.time()
      scheduleRequests()
    if time.time() > inital_time + stop_time : break #Stop getting requests if specified time is reached
    time.sleep(6)

    print("Global request queue length: ",len(requestsQueue))
  

print("Process is completed")

sum_dist = 0
sum_trips = 0
#To print average distance travelled and avg trips by the 30 vans
for van in vansQueue:
  sum_dist+=van.distance_travelled
  sum_trips+=van.trips
print("Average distance travelled by fleet is: ", (sum_dist/60))
print("Average no.of trips by fleet is: ",(sum_trips/60))

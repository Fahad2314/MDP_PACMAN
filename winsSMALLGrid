# mdpAgents.py
# parsons/20-nov-2017
#
# Version 1
#
# The starting point for CW2.
#
# Intended to work with the PacMan AI projects from:
#
# http://ai.berkeley.edu/
#
# These use a simple API that allow us to control Pacman's interaction with
# the environment adding a layer on top of the AI Berkeley code.
#
# As required by the licensing agreement for the PacMan AI we have:
#
# Licensing Information:  You are free to use or extend these projects for
# educational purposes provided that (1) you do not distribute or publish
# solutions, (2) you retain this notice, and (3) you provide clear
# attribution to UC Berkeley, including a link to http://ai.berkeley.edu.
# 
# Attribution Information: The Pacman AI projects were developed at UC Berkeley.
# The core projects and autograders were primarily created by John DeNero
# (denero@cs.berkeley.edu) and Dan Klein (klein@cs.berkeley.edu).
# Student side autograding was added by Brad Miller, Nick Hay, and
# Pieter Abbeel (pabbeel@cs.berkeley.edu).

# The agent here is was written by Simon Parsons, based on the code in
# pacmanAgents.py

from pacman import Directions
from game import Agent
import api
import random
import game
import util

class MDPAgent(Agent):

    # Constructor: this gets run when we first invoke pacman.py
    def __init__(self):
       # print "Starting up MDPAgent!"
        #name = "Pacman"
        self.pacman_steps = list()
        self.reward_map = dict()
        self.util_map =dict()
        self.flag =0
   
    def getAction(self, state):
    	  #calls the legal Actions api which returns a list of legal moves from Pac-man agents current location	
        legal = api.legalActions(state)
        #corners API returns a list of the four corners in each maze    
        corners = api.corners(state)
        #calls the API which returns wall locations 
        wall_location = api.walls(state)
        #print('walls', wall_location)
        #API call returns Pac-man agent's current location
        my_location = api.whereAmI(state)
        #API call returns food location
        food_location = api.food(state)
        #API call returns ghost locations 
        ghost_location = api.ghosts(state)
        #width of the maze
        self.width = (corners[3][0] - corners[0][0]) +1
        #height of the maze
        self.height = (corners[3][1] - corners[1][1]) +1

        #create grid of coordinates (0,0) to (6,6)
        rows = []
        for i in range(self.height):
            for j in range(self.width):
                rows.append((i,j))
        coordinates = []
        for row in rows:
            if row in wall_location:
                pass
            else:
                coordinates.append(row)

        subgrid = []   
        rewardgrid = []
        utilitygrid =[]
        #make grid 
        self.makegrid(self.height, self.width,subgrid)
        self.makegrid(self.height, self.width, rewardgrid)
        self.makegrid(self.height, self.width, utilitygrid)

        #set walls for the reward grid
        self.wallset(rewardgrid,wall_location)
        
           
        for node in coordinates:
            if node in food_location and (node not in ghost_location):
               self.reward_map[node]=10.0
            elif node in ghost_location:
                self.reward_map[node]=-10.0
            else:
                self.reward_map[node]=0.0
        
        #EACH GET STATE STARTS WITH THE UTILITY MAP AND REWARD MAP BEING IDENTICAL THIS IS TO TRACK THE MOVING PARTS CHANGE IN FOOD ETC
        self.util_map = self.reward_map
            
        

        for node in self.reward_map:
            x,y = node
            self.setValue(rewardgrid, x,y, self.reward_map[node])
        
        print 'REWARD GRID'
        print
        self.prettyDisplay(rewardgrid)

        gamma = 1
        count = 1

        while count  <= 3:
            for node in self.util_map:
                #set values in the utilitygrid for displaying
                # x,y = node 
                # self.setValue(utilitygrid, x,y, self.util_map[node])

                #iteration of values using the functions for calculated expected utility of all moves
                up = self.expected_up(node, wall_location)
                down = self.expected_down(node, wall_location)
                left = self.expected_left(node, wall_location)
                right = self.expected_right(node, wall_location)
                #list of expected utilities for each move for each node in traversable list
                expected_utilities = self.list_append(up,down,left,right)
                #manhattan distance to the ghost
                dist_to_ghost = util.manhattanDistance(node, ghost_location[0])
                #behaviour dependant on the distance to the ghost 
                if dist_to_ghost > 1:
                    self.util_map[node] = round( (-0.04 +  (gamma * max(expected_utilities))),2)
                elif dist_to_ghost == 1:
                    #expected_utilities.remove(min(expected_utilities))
                    self.util_map[node]= round( (-0.04 + (gamma * min(expected_utilities))), 2)
                elif dist_to_ghost ==0:
                    self.util_map[node]= round( (-0.04 + (gamma * min(expected_utilities))),2 )
                
                x,y = node 
                self.setValue(utilitygrid, x,y, self.util_map[node])
                

                print 'node:', node
                print 'expected utilities', expected_utilities
                print 'distance from ghost', util.manhattanDistance(node,ghost_location[0])





        
            print 'UTILITY GRID %s' % count
            print
            self.wallset(utilitygrid, wall_location)
            self.prettyDisplay(utilitygrid)
            count = count +1

        #outside of while loop where we begin each move 
        #lists makeable moves from each location pacman is at
        
        direction = self.maximum_move(my_location, wall_location,legal)

        raw_input('Press Enter')

        return api.makeMove(direction, legal)

        #return api.makeMove(Directions.STOP, legal)
        

        










    def makegrid(self,height,width,grid):
        for i in range(self.height):
            row = []
            for j in range(self.width):
                row.append(0)
            grid.append(row)

    def display(self,grid):
        for i in range(self.height):
            print grid[i]
    
    def setValue(self,grid,x, y, value):
        grid[y][x] = value
    
    def wallset(self,grid,wall_location):
        for wall in wall_location:
            x,y = wall
            self.setValue(grid, x,y,'####')

    def prettyDisplay(self,grid):       
        for i in range(self.height):
            for j in range(self.width):
                print grid[self.height - (i + 1)][j],
            print
        print

    def children(self, node):
        children = []
        x,y = node
        north = (x, y+1)
        south = (x, y-1)
        west = (x-1,y)
        east = (x+1, y)
        
        children.append(north)
        children.append(south)
        children.append(west)
        children.append(east)
       
        return children

    def makeablemoves(self, node, wall_location):
        children = []
        x,y = node
        north = (x, y+1)
        south = (x, y-1)
        west = (x-1,y)
        east = (x+1, y)
        
        if north not in wall_location:
            children.append(north)
        else:
            pass
        
        if south not in wall_location:
            children.append(south)
        else:
            pass
        if west not in wall_location:
            children.append(west)
        else:
            pass
        if east not in wall_location:
            children.append(east)
        else:
            pass
       
        return children
    
    def expected_up(self, node, wall_location):
        neighbours = self.children(node)
        up = neighbours[0]
        left = neighbours[2]
        right = neighbours[3]
        
        if up not in wall_location:
            up = round((self.util_map[up] *0.8),2)
        else:
            up = round((self.util_map[node]*0.8),2)
        
        if left not in wall_location:
            left = round((self.util_map[left] *0.1),2)
        else:
            left= round((self.util_map[node] *0.1),2)
        
        if right not in wall_location:
            right = round((self.util_map[right] *0.1),2)
        else:
            right = round((self.util_map[node] *0.1),2)
        
        final_util = round((up + left + right),2)
        return final_util
    
    def expected_down(self, node, wall_location):
        neighbours = self.children(node)
        down = neighbours[1]
        left = neighbours[2]
        right = neighbours[3]

        if down not in wall_location:
            down = round((self.util_map[down] *0.8),2)
        else:
            down = round((self.util_map[node]*0.8),2)
        
        if left not in wall_location:
            left = round((self.util_map[left] *0.1),2)
        else:
            left= round((self.util_map[node] *0.1),2)
        
        if right not in wall_location:
            right = round((self.util_map[right] *0.1),2)
        else:
            right = round((self.util_map[node] *0.1),2)
        
        final_util = round((down + left + right),2)
        return final_util
    
    def expected_left(self, node, wall_location):
        neighbours = self.children(node)
        left = neighbours[2]
        up = neighbours[0]
        down = neighbours[1]

        if left not in wall_location:
            left = round((self.util_map[left] *0.8),2)
        else:
            left = round((self.util_map[node]*0.8),2)
        
        if up not in wall_location:
            up = round((self.util_map[up] *0.1),2)
        else:
            up= round((self.util_map[node] *0.1),2)
        
        if down not in wall_location:
            down = round((self.util_map[down] *0.1),2)
        else:
            down = round((self.util_map[node] *0.1),2)
        
        final_util = round((left + up + down),2)
        return final_util

    def expected_right(self, node, wall_location):
        neighbours = self.children(node)
        right= neighbours[3]
        up = neighbours[0]
        down = neighbours[1]

        if right not in wall_location:
            right = round((self.util_map[right] *0.8),2)
        else:
            right = round((self.util_map[node]*0.8),2)
        
        if up not in wall_location:
            up = round((self.util_map[up] *0.1),2)
        else:
            up= round((self.util_map[node] *0.1),2)
        
        if down not in wall_location:
            down = round((self.util_map[down] *0.1),2)
        else:
            down = round((self.util_map[node] *0.1),2)
        
        final_util = round((right + up + down),2)
        return final_util
    
    def list_append(self,a,b,c,d):
        """ returns a list of items entered"""
        target_list = []
        target_list.append(a)
        target_list.append(b)
        target_list.append(c)
        target_list.append(d)

        return target_list
    
    def get_max_key(self,move_dict):
        'get the coordinate with the maximum utility for the next move from a dictionary of possible moves'
        list_of_values = move_dict.values()
        list_of_keys = move_dict.keys()
        index_of_key = list_of_values.index(max(list_of_values))
        max_coord = list_of_keys[index_of_key]
        
        print 'max move coordinates', max_coord
        return max_coord
    
    def pacman_moves(self,node, my_location, legal):
        'returns direction for pacman agent to traverse dependant on the adjacent node/coordinate with the highest utility'
        
        if my_location[0] < node[0]:
            if Directions.EAST in legal:
                return Directions.EAST
        
        elif my_location[0] > node[0]:
            if Directions.WEST in legal:
                return Directions.WEST
        
        elif my_location[1] > node[1]:
            if Directions.SOUTH in legal:
                return Directions.SOUTH
        
        elif my_location[1] < node[1]:
            if Directions.NORTH in legal:
                return Directions.NORTH
        
        
        



    
    
    def maximum_move(self, my_location, wall_location,legal):
        """ Returns a list of makeable moves for Pac-man then selects the move with the highest 
        expected utility according to the bellman equation"""

        makeable_moves = self.makeablemoves(my_location, wall_location)

        print 'my location', my_location
        print 'list of moves possible', makeable_moves
        #empty dict to hold the utilities of the possible moves
        move_dict = dict()

        for move in makeable_moves:
            move_dict[move]= self.util_map[move]
        
        print 'move dict', move_dict
        
        max_coord = self.get_max_key(move_dict)

        direction = self.pacman_moves(max_coord,my_location,legal)
        print('direc', direction)

        return direction

        

        
        




        
    
    

        


    




    

    




        
        
    

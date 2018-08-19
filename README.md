# test2
Apriori algorithm with association rule

# coding: utf-8
# Read csv in the form of dataframe
'''import pandas as pd
data=pd.read_csv('Apriori.csv')
#data.list <- split(data.df, seq(nrow(data.df)))
data=data.values.tolist()
data'''
# In[100]:


import csv
import sys
with open('Apriori.csv','r') as f:
  reader=csv.reader(f)
  data=list(reader)
data


# In[101]:


# to remove nan
'''for i in range(len(data)):
    data[i]=[x for x in data[i] if str(x) != 'nan']
data'''


# In[102]:


def createC1(dataSet):
    C1 = []
    for transaction in dataSet:
        for item in transaction:
            if not [item] in C1:
                C1.append([item])
                
    C1.sort()
    return list(map(frozenset, C1))#use frozen set so we
                            #can use it as a key in a dict  


# In[103]:


#D is a dataset in the setform.
D = list(map(set,data))
D


# In[104]:


def scanD(D, Ck, minSupport):
    ssCnt = {}
    for tid in D:
        for can in Ck:
            if can.issubset(tid):
                if not can in ssCnt: ssCnt[can]=1
                else: ssCnt[can] += 1
    numItems = float(len(D))
    retList = []
    supportData = {}
    ssCnt
    for key in ssCnt:
        support = ssCnt[key]#/numItems
        if support >= minSupport:
            retList.insert(0,key)
        supportData[key] = support
    return retList, supportData


# In[105]:


C1 = createC1(data)
C1


# In[106]:


L1,suppDat0 = scanD(D,C1,2)
L1


# In[107]:


def aprioriGen(Lk, k): #creates Ck
    retList = []
    lenLk = len(Lk)
    for i in range(lenLk):
        for j in range(i+1, lenLk): 
            L1 = list(Lk[i])[:k-2]; L2 = list(Lk[j])[:k-2]
            L1.sort(); L2.sort()
            if L1==L2: #if first k-2 elements are equal
                retList.append(Lk[i] | Lk[j]) #set union
    return retList


# In[108]:


def apriori(dataSet, minSupport = 2):
    C1 = createC1(dataSet)
    D = list(map(set, dataSet))
    L1, supportData = scanD(D, C1, minSupport)
    L = [L1]
    k = 2
    while (len(L[k-2]) > 0):
        Ck = aprioriGen(L[k-2], k)
        Lk, supK = scanD(D, Ck, minSupport)#scan DB to get Lk
        supportData.update(supK)
        L.append(Lk)
        k += 1
    return L, supportData


# In[109]:


L,suppData = apriori(data)
L


# In[110]:


def generateRules(L, supportData, minConf=0.6):  #supportData is a dict coming from scanD
    bigRuleList = []
    for i in range(1, len(L)):#only get the sets with two or more items
        for freqSet in L[i]:
            H1 = [frozenset([item]) for item in freqSet]
            if (i > 1):
                rulesFromConseq(freqSet, H1, supportData, bigRuleList, minConf)
            else:
                calcConf(freqSet, H1, supportData, bigRuleList, minConf)
    return bigRuleList   


# In[111]:


def calcConf(freqSet, H, supportData, brl, minConf=0.6):
    prunedH = [] #create new list to return
    for conseq in H:
        conf = supportData[freqSet]/supportData[freqSet-conseq] #calc confidence
        if conf >= minConf: 
            print (freqSet-conseq,'-->',conseq,'conf:',conf)
            brl.append((freqSet-conseq, conseq, conf))
            prunedH.append(conseq)
    return prunedH


# In[112]:


def rulesFromConseq(freqSet, H, supportData, brl, minConf=0.6):
    m = len(H[0])
    if (len(freqSet) > (m + 1)): #try further merging
        Hmp1 = aprioriGen(H, m+1)#create Hm+1 new candidates
        Hmp1 = calcConf(freqSet, Hmp1, supportData, brl, minConf)
        if (len(Hmp1) > 1):    #need at least two sets to merge
            rulesFromConseq(freqSet, Hmp1, supportData, brl, minConf)


# In[113]:


L,suppData= apriori(data,minSupport=2)
L


# In[114]:


rules= generateRules(L,suppData, minConf=0.6)

### implementation 

{
 "cells": [
  {
   "cell_type": "raw",
   "metadata": {},
   "source": [
    "# Read csv in the form of dataframe\n",
    "'''import pandas as pd\n",
    "data=pd.read_csv('Apriori.csv')\n",
    "#data.list <- split(data.df, seq(nrow(data.df)))\n",
    "data=data.values.tolist()\n",
    "data'''"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 49,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "[['i1', 'i2', 'i5'],\n",
       " ['i2', 'i4'],\n",
       " ['i2', 'i3'],\n",
       " ['i1', 'i2', 'i4'],\n",
       " ['i1', 'i3'],\n",
       " ['i2', 'i3'],\n",
       " ['i1', 'i3'],\n",
       " ['i1', 'i2', 'i3', 'i5'],\n",
       " ['i1', 'i2', 'i3']]"
      ]
     },
     "execution_count": 49,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "import csv\n",
    "import sys\n",
    "with open('Apriori.csv','r') as f:\n",
    "  reader=csv.reader(f)\n",
    "  data=list(reader)\n",
    "data"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 50,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "\"for i in range(len(data)):\\n    data[i]=[x for x in data[i] if str(x) != 'nan']\\ndata\""
      ]
     },
     "execution_count": 50,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# to remove nan\n",
    "'''for i in range(len(data)):\n",
    "    data[i]=[x for x in data[i] if str(x) != 'nan']\n",
    "data'''"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 53,
   "metadata": {},
   "outputs": [],
   "source": [
    "def createC1(dataSet):\n",
    "    C1 = []\n",
    "    for transaction in dataSet:\n",
    "        for item in transaction:\n",
    "            if not [item] in C1:\n",
    "                C1.append([item])\n",
    "                \n",
    "    C1.sort()\n",
    "    return list(map(frozenset, C1))#use frozen set so we\n",
    "                            #can use it as a key in a dict  "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 42,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "[{'i1', 'i2', 'i5'},\n",
       " {'i2', 'i4'},\n",
       " {'i2', 'i3'},\n",
       " {'i1', 'i2', 'i4'},\n",
       " {'i1', 'i3'},\n",
       " {'i2', 'i3'},\n",
       " {'i1', 'i3'},\n",
       " {'i1', 'i2', 'i3', 'i5'},\n",
       " {'i1', 'i2', 'i3'}]"
      ]
     },
     "execution_count": 42,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "#D is a dataset in the setform.\n",
    "D = list(map(set,data))\n",
    "D"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 54,
   "metadata": {},
   "outputs": [],
   "source": [
    "def scanD(D, Ck, minSupport):\n",
    "    ssCnt = {}\n",
    "    for tid in D:\n",
    "        for can in Ck:\n",
    "            if can.issubset(tid):\n",
    "                if not can in ssCnt: ssCnt[can]=1\n",
    "                else: ssCnt[can] += 1\n",
    "    numItems = float(len(D))\n",
    "    retList = []\n",
    "    supportData = {}\n",
    "    ssCnt\n",
    "    for key in ssCnt:\n",
    "        support = ssCnt[key]#/numItems\n",
    "        if support >= minSupport:\n",
    "            retList.insert(0,key)\n",
    "        supportData[key] = support\n",
    "    return retList, supportData"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 55,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "[frozenset({'i1'}),\n",
       " frozenset({'i2'}),\n",
       " frozenset({'i3'}),\n",
       " frozenset({'i4'}),\n",
       " frozenset({'i5'})]"
      ]
     },
     "execution_count": 55,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "C1 = createC1(data)\n",
    "C1"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 56,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "[frozenset({'i3'}),\n",
       " frozenset({'i4'}),\n",
       " frozenset({'i5'}),\n",
       " frozenset({'i2'}),\n",
       " frozenset({'i1'})]"
      ]
     },
     "execution_count": 56,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "L1,suppDat0 = scanD(D,C1,2)\n",
    "L1"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 57,
   "metadata": {},
   "outputs": [],
   "source": [
    "def aprioriGen(Lk, k): #creates Ck\n",
    "    retList = []\n",
    "    lenLk = len(Lk)\n",
    "    for i in range(lenLk):\n",
    "        for j in range(i+1, lenLk): \n",
    "            L1 = list(Lk[i])[:k-2]; L2 = list(Lk[j])[:k-2]\n",
    "            L1.sort(); L2.sort()\n",
    "            if L1==L2: #if first k-2 elements are equal\n",
    "                retList.append(Lk[i] | Lk[j]) #set union\n",
    "    return retList"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 58,
   "metadata": {},
   "outputs": [],
   "source": [
    "def apriori(dataSet, minSupport = 2):\n",
    "    C1 = createC1(dataSet)\n",
    "    D = list(map(set, dataSet))\n",
    "    L1, supportData = scanD(D, C1, minSupport)\n",
    "    L = [L1]\n",
    "    k = 2\n",
    "    while (len(L[k-2]) > 0):\n",
    "        Ck = aprioriGen(L[k-2], k)\n",
    "        Lk, supK = scanD(D, Ck, minSupport)#scan DB to get Lk\n",
    "        supportData.update(supK)\n",
    "        L.append(Lk)\n",
    "        k += 1\n",
    "    return L, supportData"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 48,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "[[frozenset({'i3'}),\n",
       "  frozenset({'i4'}),\n",
       "  frozenset({'i5'}),\n",
       "  frozenset({'i2'}),\n",
       "  frozenset({'i1'})],\n",
       " [frozenset({'i1', 'i3'}),\n",
       "  frozenset({'i2', 'i3'}),\n",
       "  frozenset({'i2', 'i4'}),\n",
       "  frozenset({'i1', 'i2'}),\n",
       "  frozenset({'i1', 'i5'}),\n",
       "  frozenset({'i2', 'i5'})],\n",
       " [frozenset({'i1', 'i2', 'i3'}), frozenset({'i1', 'i2', 'i5'})],\n",
       " []]"
      ]
     },
     "execution_count": 48,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "L,suppData = apriori(data)\n",
    "L"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 59,
   "metadata": {},
   "outputs": [],
   "source": [
    "def generateRules(L, supportData, minConf=0.6):  #supportData is a dict coming from scanD\n",
    "    bigRuleList = []\n",
    "    for i in range(1, len(L)):#only get the sets with two or more items\n",
    "        for freqSet in L[i]:\n",
    "            H1 = [frozenset([item]) for item in freqSet]\n",
    "            if (i > 1):\n",
    "                rulesFromConseq(freqSet, H1, supportData, bigRuleList, minConf)\n",
    "            else:\n",
    "                calcConf(freqSet, H1, supportData, bigRuleList, minConf)\n",
    "    return bigRuleList   "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 60,
   "metadata": {},
   "outputs": [],
   "source": [
    "def calcConf(freqSet, H, supportData, brl, minConf=0.6):\n",
    "    prunedH = [] #create new list to return\n",
    "    for conseq in H:\n",
    "        conf = supportData[freqSet]/supportData[freqSet-conseq] #calc confidence\n",
    "        if conf >= minConf: \n",
    "            print (freqSet-conseq,'-->',conseq,'conf:',conf)\n",
    "            brl.append((freqSet-conseq, conseq, conf))\n",
    "            prunedH.append(conseq)\n",
    "    return prunedH"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 61,
   "metadata": {},
   "outputs": [],
   "source": [
    "def rulesFromConseq(freqSet, H, supportData, brl, minConf=0.6):\n",
    "    m = len(H[0])\n",
    "    if (len(freqSet) > (m + 1)): #try further merging\n",
    "        Hmp1 = aprioriGen(H, m+1)#create Hm+1 new candidates\n",
    "        Hmp1 = calcConf(freqSet, Hmp1, supportData, brl, minConf)\n",
    "        if (len(Hmp1) > 1):    #need at least two sets to merge\n",
    "            rulesFromConseq(freqSet, Hmp1, supportData, brl, minConf)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 63,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "[[frozenset({'i3'}),\n",
       "  frozenset({'i4'}),\n",
       "  frozenset({'i5'}),\n",
       "  frozenset({'i2'}),\n",
       "  frozenset({'i1'})],\n",
       " [frozenset({'i1', 'i3'}),\n",
       "  frozenset({'i2', 'i3'}),\n",
       "  frozenset({'i2', 'i4'}),\n",
       "  frozenset({'i1', 'i2'}),\n",
       "  frozenset({'i1', 'i5'}),\n",
       "  frozenset({'i2', 'i5'})],\n",
       " [frozenset({'i1', 'i2', 'i3'}), frozenset({'i1', 'i2', 'i5'})],\n",
       " []]"
      ]
     },
     "execution_count": 63,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "L,suppData= apriori(data,minSupport=2)\n",
    "L"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 64,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "frozenset({'i3'}) --> frozenset({'i1'}) conf: 0.6666666666666666\n",
      "frozenset({'i1'}) --> frozenset({'i3'}) conf: 0.6666666666666666\n",
      "frozenset({'i3'}) --> frozenset({'i2'}) conf: 0.6666666666666666\n",
      "frozenset({'i4'}) --> frozenset({'i2'}) conf: 1.0\n",
      "frozenset({'i1'}) --> frozenset({'i2'}) conf: 0.6666666666666666\n",
      "frozenset({'i5'}) --> frozenset({'i1'}) conf: 1.0\n",
      "frozenset({'i5'}) --> frozenset({'i2'}) conf: 1.0\n",
      "frozenset({'i5'}) --> frozenset({'i1', 'i2'}) conf: 1.0\n"
     ]
    }
   ],
   "source": [
    "rules= generateRules(L,suppData, minConf=0.6)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": []
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.6.5"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 2
}



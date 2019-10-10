import requests
import bs4 
from nltk.tokenize import sent_tokenize, word_tokenize
from nltk.corpus import stopwords
from collections import defaultdict
from string import punctuation
from heapq import nlargest

class FrequencySummarizer:
    def __init__(self,min_cut=0.1,max_cut=0.9):
        #__init__ is a contructor and the function will be called each time an object is created/instantiated
        self._min_cut = min_cut
        self._max_cut = max_cut
        #The variables above are member variabes since there are assigned to 'self' as well as an _ as the first char
        self._stopwords = set(stopwords.words('english') + list(punctuation))
    def _compute_frequencies(self, word_sent):
        #This method will take in a list of sentences, and output a dictionary wher the keys are words, and the values 
        #are frequencies of those words in the set of sentences
        freq = defaultdict(int)
        #defaultdict is a class that inherits from dictionary which prevents KeyError from occuring
        for s in word_sent:
            for word in s:
                if word not in self._stopwords:
                    freq[word] += 1
                    #These 2 loops take every word in every sentence, and keep track of its frequency but only to non-stop words
        #Normalize the frequencies by dividing each by the highest frequency. This helps to make frequencies comparable since
        #they will all be between 0 and 1. 
        #Filter out frequencies that are too high or too low. Helps to eliminate 'almost stopwords' from the list of stopwords
        m = float(max(freq.values()))
        for w in list(freq):
            freq[w] = freq[w]/m
            if freq[w] >= self._max_cut or freq[w] <= self._min_cut:
                del freq[w]
        return freq
    def summarize(self,text,n):
        sents = sent_tokenize(text)
        #split text into a list of sentences
        assert n <= len(sents)
        #'assert' is used to make sure a condition hold true, else an exception is thrown. This specific code makes sure
        #that the summary isn't longer the the article
        word_sent = [word_tokenize(s.lower()) for s in sents]
        self._freq = self._compute_frequencies(word_sent)
        ranking = defaultdict(int)
        for i,sent in enumerate(word_sent):
            #This for loop is different since it uses a built-in function called enumerate and gains a list of tuples
            for w in sent: 
                if w in self._freq:
                    ranking[i] += self._freq[w]
        #Each word in each sentence computed a rank for that sentence as the sum of the frequencies of the words in 
        #that sentence
        sents_idx = nlargest(n,ranking,key = ranking.get)
        #the nlargest function requires to know how to sort the sentences so it needs to be a given a funciton that it
        #can use to compare 2 sentences
        return [sents[j] for j in sents_idx]

import urllib.request
from bs4 import BeautifulSoup

someUrl = "https://en.wikipedia.org/wiki/C%2B%2B"



def get_only_text_url_2(url):
    page = urllib.request.urlopen(url).read().decode('utf8') 
    soup = BeautifulSoup(page, 'html.parser')
    text = ' '.join(map(lambda p: p.text,soup.find_all('#firstHeading')))
    soup2 = BeautifulSoup(text, 'html.parser')
    text = ' '.join(map(lambda p: p.text, soup.find_all('p')))
    return soup.title.text , text


textOfUrl = get_only_text_url_2(someUrl)
fs = FrequencySummarizer()
summary = fs.summarize(textOfUrl[1],3)


print(summary) 

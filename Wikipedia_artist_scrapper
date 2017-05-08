########################################################################
## Michael Puthawala
## michaelputhawala@gmail.com
## 2/9/2017
######################################################################## 


# Module used to get the HTML
import requests

# Module used for regular expression matching
import re

import hashlib

import os

# Module used to save the data to a csv, in retrospect this module is probably a bad idea
import csv

# Module used to parse the HTML
from bs4 import BeautifulSoup

#global INFO_CARD_CLASS = ['infobox biography vcard',]

global OUTPUT_DELIMITOR

global BAD_PATTERNS

BAD_PATTERNS = re.compile("^\[[0-9]+\]$")

OUTPUT_DELIMITOR = ';'


def reccursive_extract_string(tag):
    "Gets all of the strings associated with a tree"
    r = []
    if tag is None:
        return r
    tag_cont = tag.contents
    for content_piece in tag_cont:
        if isinstance(content_piece,str):
            r.append(content_piece)
        else:
            r = r + reccursive_extract_string(content_piece)
    return r

def scrape_name_from_row(table_row):
    "Given a table row, pull the name of the artist"
    name_box = table_row.find('span', class_=re.compile('^fn'))
    return clean_string({"name": reccursive_extract_string(name_box)})

def clean_string (str_dict):
    ""
    r = {}
    for key in str_dict:
        # For every value stored in the dictionary (which is a list)
        cleaned_list = []
        for dirty_string in str_dict[key]:
            # First, separate anything in the list which might have commas in it
            for clean_string in dirty_string.split(','):
                # then make sure that what we are adding actually contains interesting characters
                clean_string = clean_string.replace('\n','').replace('\t','').replace('\u2013',' ')
                if re.search(r'[\w]',clean_string) and not BAD_PATTERNS.match(clean_string):
                    cleaned_list.append(clean_string)
        r[key] = cleaned_list

    return r

def scrape_origin_from_row(table_row):
    "Given a table row, pull the origin of the artist"
    origin_list = table_row.find('td')
    return clean_string({"origin" : reccursive_extract_string(origin_list)})

def scrape_genres_from_row(table_row):
    "Given a table row, pull the genres associated with the artist"
    genre_list = table_row.find('td')
    return clean_string({"genres" : reccursive_extract_string(genre_list)})

def scrape_years_active_from_row(table_row):
    "Given a table row, pull the years active of the artist"
    years_active_list = table_row.find('td')
    return clean_string({"years active" : reccursive_extract_string(years_active_list)})

def scrape_labels_from_row(table_row):
    "Given a table row, pull the labels of the artist"
    label_list = table_row.find('td')
    return clean_string({"labels" : reccursive_extract_string(label_list)})

    labels = []
    for list_el in table_row.find_all('li'):
        labels.append(list_el.find('a').string)
    if len(labels) is not 0:
        return clean_string({"labels" : labels})
    else:
        # try and see of the labels are in a neighboring tag
        maybe_label = table_row.find('td')
        if maybe_label is None:
            return None
        labels = reccursive_extract_string(maybe_label)
        return clean_string({"labels" : labels})

def scrape_ass_artists_from_row(table_row):
    "Given a table row, pull the names of the associated artists of that artist"
    asscociated_acts = []
    for list_el in table_row.find_all('li'):
        tmp_attr = list_el.find('a')
        if tmp_attr is not None:
            if tmp_attr.string is not None:
                asscociated_acts.append(tmp_attr.string)
            else:
                continue
        else:
            asscociated_acts.append(list_el.string)
    if len(asscociated_acts) is not 0:
        return clean_string({"associated acts" : asscociated_acts})
    else:
        # try and see of the genres are in a neighboring tag
        maybe_act = table_row.find('td')
        return clean_string({"associated acts" : reccursive_extract_string(maybe_act)})

def scrape_intruments_from_row(table_row):
    "Given a table row, pull the instruments of the artist"
    intruments_list = table_row.find('td')
    return clean_string({"instruments" : reccursive_extract_string(intruments_list)})

def scrape_also_known_as_from_row(table_row):
    "Given a table row, pull the alises of the artist"
    aka_list = table_row.find('td')
    return clean_string({"also known as" : reccursive_extract_string(aka_list)})

def scrape_members_from_row(table_row):
    "Given a table row, pull the current members of the band"
    member_list = table_row.find('td')
    return clean_string({"members" : reccursive_extract_string(member_list)})

def scrape_past_members_from_row(table_row):
    "Given a table row, pull the former members of the band"
    past_member_list = table_row.find('td')
    return clean_string({"past members" : reccursive_extract_string(past_member_list)})

def table_row_switch_dict(table_header):
    return{
        "origin" : scrape_origin_from_row,
        "genres" : scrape_genres_from_row,
        "years active" : scrape_years_active_from_row,
        "labels" : scrape_labels_from_row,
        "associated acts" : scrape_ass_artists_from_row,
        "instruments" : scrape_intruments_from_row,
        "members" : scrape_members_from_row,
        "past members" : scrape_past_members_from_row,
        "also known as" : scrape_also_known_as_from_row,
        }.get(table_header,None)


def determine_infocard_row_content(table_row):
    "Given a table row, figure out if we want to scrape the data contained in that row"
    # is it the name of the artist?
    if table_row.find('span', class_=re.compile('^fn')) is not None:
        return scrape_name_from_row

    if table_row.find('th') is not None:
        table_header = table_row.find('th')
        # Either the header contains the string directly, or it's wrapped in a span
        name_of_column_info = ''
        if table_header.string is not None:
            name_of_column_info = table_header.string
        else:
            # is it wrapped in a span?
            if table_header.find('span') is not None:
                name_of_column_info = table_header.find('span').string
            else:
                return None

        if name_of_column_info is None:
            return None
        return table_row_switch_dict(name_of_column_info.lower())

def determine_infocard_row_name(table_row):
    "Given a table row, figure out the name of the row"
    # is it the name of the artist?
    if table_row.find('span', class_=re.compile('^fn')) is not None:
        return "name"

    if table_row.find('th') is not None:
        table_header = table_row.find('th')
        # Either the header contains the string directly, or it's wrapped in a span
        name_of_column_info = ''
        if table_header.string is not None:
            name_of_column_info = table_header.string
        else:
            # is it wrapped in a span?
            if table_header.find('span') is not None:
                name_of_column_info = table_header.find('span').string
            else:
                return None

        if name_of_column_info is None:
            return None
        return name_of_column_info.lower()

# this function scrapes the personal details off of one wiki page, provided that it is formatted correctly.
def scrape_person_page(url):
    "This scrapes biological information from a wiki page."
    # first, get the HTML, and soup it
    #print('Getting html from ' + url)

    response = requests.get(url)

    
    url_hash_object = hashlib.md5(response.url.encode())
    url_hash = url_hash_object.hexdigest()
    #print('Getting html from ' + url)
    #print('Looking for into card')
    soup = BeautifulSoup(response.content,'html.parser')

    # look for all tables whose classes start with infobox.

    info_card = soup.find('table',class_=re.compile("^infobox"))

    if info_card is None:
        print("Couldn't find an info card")
        return
    r = {"ID" : url_hash}
    
    for table_row in info_card.find_all('tr'):
        row_scrape_resp = determine_infocard_row_content(table_row)
        row_name = determine_infocard_row_name(table_row)
        if row_scrape_resp is None:
            continue
        scrapped_row_info = row_scrape_resp(table_row)
        if scrapped_row_info is not None:
            r[row_name] = scrapped_row_info[row_name]

    #print(r)


    return r

def scrape_artist_url_from_landing_page(url):
    "This function is given a base page (e.g. https://en.wikipedia.org/wiki/List_of_jazz_musicians) and returns a list. Each element of that list is the URL of a artist listed on that base page."
    # first, get the HTML, and soup it
    print('Getting html from ' + url)
    response = requests.get(url)
    print('Getting html from ' + url)
    print('Looking for into card')
    soup = BeautifulSoup(response.content,'html.parser')

    # find the main content text

    mw_content_text = soup.find('div',id='mw-content-text')
    
    r = []

    for link in mw_content_text.find_all('a',href=re.compile('^/wiki/')):
        r.append(link['href']);

    return r

def find_all_entries_between(list,first_val_to_add, last_val_to_add):
    "Returns all of the elements in list that are between (including endpoints) the second and third arguments."
    add_el_to_return = False;
    r = []

    for el in list:
        if el == first_val_to_add:
            add_el_to_return = True
        if add_el_to_return:
            r.append(el)
        if el == last_val_to_add:
            add_el_to_return = False
    return r

def scrape_genre(file_name,name,genre_list_URL,first_URL,last_URL):
    # the home page of wiki
    homepage = 'https://en.wikipedia.org'

    # This is the base page, which has the list of artists on it
    base_page_URL = homepage+genre_list_URL
    
    # this function looks at the base page, and looks through it to find things that look like URLs to pages of possible artists
    possible_person_url = scrape_artist_url_from_landing_page(base_page_URL)

    # a lot of the URLS on possible_person_URL don't actually belong to artists. This function trims that list to reduce the amount of time you waste
    person_urls = find_all_entries_between(possible_person_url,first_URL,last_URL)

    # make a path for the new information
    genre_dir_path = './'+name+'/'

    if not os.path.exists(genre_dir_path ):
        dir = os.mkdir(genre_dir_path )
    

    # dictionary containing the file streams of all of the output files
    output_files_dict = {}

   # person_urls = ['/wiki/Epica_(band)']

    for person_URL in person_urls:
        #try:
        #    # Try and see if you can scrape the information about the artist on that page
        #    person_facts = scrape_person_page(homepage + person_URL)

        #    # If you can't just continue. Don't crash the whole program
        #except AttributeError:
        #    print('AttributeError while scrapping')
        person_facts = scrape_person_page(homepage + person_URL)
        if person_facts is None:
            continue
        ID = person_facts["ID"]

        # go though each of the scrapped keys, find that file, and 
        for fact_key in person_facts:
            if fact_key == "ID":
                continue

            # if there is a fstream belonging to that artist already
            if fact_key not in output_files_dict:
                output_files_dict[fact_key] = open(genre_dir_path + fact_key + '.csv','w')

            for factoid in person_facts[fact_key]:
                try:
                    output_files_dict[fact_key].write(ID + OUTPUT_DELIMITOR + factoid + '\n')
                    # If you can't encode a character, just continue onwards.
                except UnicodeEncodeError:
                    continue


    for output_file_key in output_files_dict:
        output_files_dict[output_file_key].close()


    return


# this is the rpefix of the file where the genre information will be saved
genre_file_name = 'inti_pop_pt2'

# this is the name of the genre as it will be displayed in the csv
genre_name = 'Indi Pop'

# this is the URL of the list which you would like to scrape from
genre_list_URL = '/wiki/List_of_indie_pop_artists'

# this is the url of the first artist that you would like to scrape
genre_first_URL = '/wiki/Sodagreen'

# this is the url of the last artist that you would like to scrape
genre_last_URL = '/wiki/Zooey_Deschanel'

# this function does all of the magic
# scrape_genre(genre_file_name,genre_name,genre_list_URL,genre_first_URL,genre_last_URL)

#hash_object = hashlib.md5(b'Hello World')
#hex_dig = hash_object.hexdigest()
#print(str(int(hex_dig,16)))

#str = "a ,b c,b,d,e,f,y,asfadsgargerg,,sdg"
#print(str.split(','))

#d = {'past members': ['Yves Huts', '\n', 'Ad Sluijter', '\n', 'Helena Michaelsen', '\nJeroen Simons']}

#print(clean_string(d))





scrape_person_page('https://en.wikipedia.org/wiki/Epica_(band)')

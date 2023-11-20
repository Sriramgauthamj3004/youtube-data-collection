# youtube-data-collection
This Project is based on Youtube Data Harvesting.Collecting a youtube datas using channel ids and collect &store the datas of youtube channels.
In this project the users we can easily able to get the information about the youtube channels through channel id. We are analyse the data of the channel using MONGODB
and SQL and STREAMLIT are used in this project to make the user so friendly to collect the datas about the youtube channels and the user can retrive ,save and query the youtube channel and video datas.

TOOLS USED IN PROJECT :

JUPYTER :
This is a best platform to build the python program for developers to develop the python language. It is very user friendly to import the libraries and install , this platform is make easy way to run the python program with your incredible python skills.

PYTHON:
 Python is a more powerful language to develop the anything and everything.Python is very easy to learn and understand, it is a primary language to employed for developing for compelete application including data retrival and analysis and visualizaton also. This python language is easily understandable for machines and alos for humans.

 GOOGLEAPI CLIENT :
 The Google api client is key for communicating the data for the google application using the API.Every user have a individual api keys,it is a primary purpose in this project to interact with youtube data API V3,allowing to collect the youtube datas of information like channel details , video details and informations, playlist informations and comments details.
 By using api key the pyhton developer can easily to fetch a data of youtube and manipulate a youtube extensive data conver through a code.

 MONGODB:
 MongoDB is built on a scale out architecture that has popular with developers all kind of developing data into data schemas.As a database of document,Mongodb is easily for developers any type of data to be store in structure or unstructure data in json method. It uses a json like format to store the documents. Json is Javascript Object Notation it is used to transmitting data in web application.

 POSTGRESQL:
 Postgresql is a open source, advanced and highly scalable databasse management system(DBMS)know it realiablity and extensive features It provides a platform for storing a data of in table form and managing structured data , supporting a various data types and advanced SQL features.

 YOUTUBE DATA SCRAPPING:
 When engaging in the scrapping the youtube data content its so crucial to approach in more ethical and responsible way. Respecting the terms and youtube conditions ,obtaining appropraite the organization and ahering the data protection regulation are fundatamental considerations. The collected dat must be handled in responsibility to ensure the privacy and preventing any misuse or misrepresentation.Futhurmore it is more imporatnt to take into the account the potential impact on this platform and scrapping process. we can uphold the while extracting the data from the youtube data.

 REQURIED LIBARIES:
 1.Googleapiclient.discovery
 2.pymongo
 3.pandas
 4.streamlit
 5.psycopg2

 FEATURES:
YOUTUBE DATA HARVESTING: This project is used for collecting the channel id from the youtube channel using the api key.Then gettting a channel id and making a code for extracting a dta using the python language get the inforamtion of channel regarding channels information and video information using the youtube api.

Storage of data in MONGODB database as a data collections or data list.
Migartion of data from the data collections in mongodb and convert into tabular form to a SQL database for effiecient querying and analysis.
Search and retriveal of data from the sql data base using different search options

from googleapiclient.discovery import build
import pymongo
import psycopg2
import pandas as pd
import streamlit as st

#API KEY CONNECTION

def Api_connect():
    Api_Id="AIzaSyA3q5bk8hDwoLryx6ZvioCPQd1JmA6pWGM"

    api_service_name="youtube"
    api_version="v3"

    youtube=build(api_service_name,api_version,developerKey=Api_Id)

    return youtube

youtube=Api_connect()


#GET CHANNAL INFORMATION
def get_channel_info(channel_id):
    request=youtube.channels().list(
                part="snippet,contentDetails,statistics",
                id=channel_id
    )
    response=request.execute()

    for i in response['items']:
        data=dict(Channel_Name=i["snippet"]["title"],
                 Channel_Id=i["id"],
                 Subscribers=i["statistics"]["subscriberCount"],
                 Views=i["statistics"]["viewCount"],
                  Total_Videos=i["statistics"]["videoCount"],
                  Channel_Description=i["snippet"]["description"],
                  Playlist_Id=i["contentDetails"]["relatedPlaylists"]["uploads"])
    return data   
       
#get video ids
def get_videos_ids(channel_id):
    video_ids=[]
    response=youtube.channels().list(id=channel_id,
                                    part='contentDetails').execute()
    Playlist_Id=response['items'][0]['contentDetails']['relatedPlaylists']['uploads']

    next_page_token=None

    while True:
        response1=youtube.playlistItems().list(
                                            part='snippet',
                                            playlistId=Playlist_Id,
                                            maxResults=50,
                                            pageToken=next_page_token).execute()
        for i in range(len(response1['items'])):
            video_ids.append(response1['items'][i]['snippet']['resourceId']['videoId'])
        next_page_token=response1.get('nextPageToken')

        if next_page_token is None:
            break
    return video_ids


#get video information
def get_video_info(video_ids):
    video_data=[]
    for video_id in video_ids:
        request=youtube.videos().list(
            part="snippet,ContentDetails,statistics",
            id=video_id
        )
        response=request.execute()

        for item in response["items"]:
            data=dict(Channel_Name=item['snippet']['channelTitle'],
                      channel_Id=item['snippet']['channelId'],
                      Video_Id=item['id'],
                      Title=item['snippet']['title'],
                      Tags=item['snippet'].get('tags'),
                      Thumbnail=item['snippet']['thumbnails']['default']['url'],
                      Description=item['snippet'].get('description'),
                      Published_date=item['snippet']['publishedAt'],
                      Duration=item['contentDetails']['duration'],
                      Views=item['statistics'].get('viewCount'),
                      Likes=item['statistics'].get('likeCount'),
                      Comments=item['statistics'].get('commentCount'),
                      Favorite_count=item['statistics']['favoriteCount'],
                      Definition=item['contentDetails']['definition'],
                      Caption_status=item['contentDetails']['caption']
                     )
            video_data.append(data)
    return video_data
       

#get comment information
def get_comment_info(video_Ids):
    comment_data=[]
    try:
        for video_id in video_Ids:
            request=youtube.commentThreads().list(
                part="snippet",
                videoId= video_id,
                maxResults=50
            )
            response=request.execute()

            for item in response['items']:
                data=dict(comment_Id=item['snippet']['topLevelComment']['id'],
                          video_Id=item['snippet']['topLevelComment']['snippet']['videoId'],
                          Comment_Text=item['snippet']['topLevelComment']['snippet']['textDisplay'],
                          Comment_Author=item['snippet']['topLevelComment']['snippet']['authorDisplayName'],
                          Comment_published=item['snippet']['topLevelComment']['snippet']['publishedAt'])

                comment_data.append(data)

    except:
        pass
    return comment_data


#get_playlist_details

def get_playlist_details(channel_id):
        next_page_token=None
        All_data=[]
        while True:
                request=youtube.playlists().list(
                        part='snippet,contentDetails',
                        channelId=channel_id,
                        maxResults=50,
                        pageToken=next_page_token
                )
                response=request.execute()

                for item in response['items']:
                        data=dict(Playlist_Id=item['id'],
                                  Title=item['snippet']['title'],
                                  Channel_Id=item['snippet']['channelId'],
                                  Channel_Name=item['snippet']['channelTitle'],
                                  PublishedAt=item['snippet']['publishedAt'],
                                  Video_Count=item['contentDetails']['itemCount'])
                        All_data.append(data)

                next_page_token=response.get('nextPageToken')
                if next_page_token is None:
                        break
        return All_data


#UPLOAD TO MONGODB
mongodb_url="mongodb://localhost:27017"
client=pymongo.MongoClient(mongodb_url)
db_name = "Youtube_Data" 
db = client[db_name]


def channel_details(channel_id):
    ch_details=get_channel_info(channel_id)
    vi_ids=get_videos_ids(channel_id)
    vi_details=get_video_info(vi_ids)
    com_details=get_comment_info(vi_ids)
    pl_details=get_playlist_details(channel_id)
    
    
    coll1=db["channel_details"]
    coll1.insert_one({"channel_information":ch_details,"playlist_information":pl_details,
                     "viedo_information":vi_details,"comment_information":com_details})
    
    return "upload completed successfully"


#Table creation for channels,playlist,viedos,comments
def channels_table():
    mydb=psycopg2.connect(host="localhost",
                          user="postgres",
                          password=1999,
                          database="Youtubedata",
                          port="5432")
    cursor=mydb.cursor()

    drop_query='''drop table if exists channels'''
    cursor.execute(drop_query)
    mydb.commit()

    try:
        create_query='''create table if not exists channels(Channel_Name varchar(2000),
                                                            Channel_Id varchar(2000) primary key,
                                                            Subscribers bigint,
                                                            Views bigint,
                                                            Total_Videos int,
                                                            Channel_Description text,
                                                            Playlist_Id varchar(1000))'''

        cursor.execute(create_query)
        mydb.commit()
    except:
        print("channels table already created")


    ch_list=[]
    db=client["Youtube_Data"]
    coll1=db["channel_details"]
    for ch_data in coll1.find({},{"_id":0,"channel_information":1}):
        ch_list.append(ch_data["channel_information"])
    df=pd.DataFrame(ch_list)   




    for index,row in df.iterrows():
        insert_query='''insert into channels(Channel_Name,
                                             Channel_Id,
                                             Subscribers,
                                             Views,
                                             Total_Videos,
                                             Channel_Description,
                                             Playlist_Id)

                                             values(%s,%s,%s,%s,%s,%s,%s)'''
        values=(row['Channel_Name'],
                row['Channel_Id'],
                row['Subscribers'],
                row['Views'],
                row['Total_Videos'],
                row['Channel_Description'],
                row['Playlist_Id'])

        try:
            cursor.execute(insert_query,values)
            mydb.commit()

        except:
            print("channels values are already inserted")


def playlist_table():
    mydb=psycopg2.connect(host="localhost",
                          user="postgres",
                          password=1999,
                          database="Youtubedata",
                          port="5432")
    cursor=mydb.cursor()

    drop_query='''drop table if exists playlists'''
    cursor.execute(drop_query)
    mydb.commit()


    create_query='''create table if not exists playlists(Playlist_Id varchar(2000)primary key,
                                                            Title varchar(2000),
                                                            Channel_Id varchar(2000),
                                                            Channel_Name varchar(2000),
                                                            PublishedAt timestamp,
                                                            Video_Count int)'''



    cursor.execute(create_query)
    mydb.commit()

    
    pl_list=[]
    db=client["Youtube_Data"]
    coll1=db["channel_details"]
    for pl_data in coll1.find({},{"_id":0,"playlist_information":1}):
        for i in range(len(pl_data["playlist_information"])):
            pl_list.append(pl_data["playlist_information"][i])
    df1=pd.DataFrame(pl_list)
    
    for index,row in df1.iterrows():
        insert_query='''insert into playlists(Playlist_Id,
                                             Title,
                                             Channel_Id,
                                             Channel_Name,
                                             PublishedAt,
                                             Video_Count
                                             )

                                             values(%s,%s,%s,%s,%s,%s)'''
        values=(row['Playlist_Id'],
                row['Title'],
                row['Channel_Id'],
                row['Channel_Name'],
                row['PublishedAt'],
                row['Video_Count'])
                

        
        cursor.execute(insert_query,values)
        mydb.commit()
    

def videos_table():
    mydb=psycopg2.connect(host="localhost",
                          user="postgres",
                          password=1999,
                          database="Youtubedata",
                          port="5432")
    cursor=mydb.cursor()

    drop_query='''drop table if exists videos'''
    cursor.execute(drop_query)
    mydb.commit()


    create_query='''create table if not exists videos(Channel_Name varchar(2000),
                                                      channel_Id varchar(2000),
                                                      Video_Id varchar(2000) primary key,
                                                      Title varchar(5000),
                                                      Tags text,
                                                      Thumbnail varchar(5000),
                                                      Description text,
                                                      Published_date timestamp,
                                                      Duration interval,
                                                      Views bigint,
                                                      Likes bigint,
                                                      Comments int,
                                                      Favorite_count int,
                                                      Definition varchar(1000),
                                                      Caption_status varchar(500)
                                                      )'''



    cursor.execute(create_query)
    mydb.commit()


    vi_list=[]
    db=client["Youtube_Data"]
    coll1=db["channel_details"]
    for vi_data in coll1.find({},{"_id":0,"viedo_information":1}):
        for i in range(len(vi_data["viedo_information"])):
            vi_list.append(vi_data["viedo_information"][i])
    df2=pd.DataFrame(vi_list)


    for index,row in df2.iterrows():
            insert_query='''insert into videos(Channel_Name,
                                                  channel_Id,
                                                  Video_Id,
                                                  Title,
                                                  Tags,
                                                  Thumbnail,
                                                  Description,
                                                  Published_date,
                                                  Duration,
                                                  Views,
                                                  Likes,
                                                  Comments,
                                                  Favorite_count,
                                                  Definition,
                                                  Caption_status
                                                 )

                                                 values(%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)'''
            values=(row['Channel_Name'],
                    row['channel_Id'],
                    row['Video_Id'],
                    row['Title'],
                    row['Tags'],
                    row['Thumbnail'],
                    row['Description'],
                    row['Published_date'],
                    row['Duration'],
                    row['Views'],
                    row['Likes'],
                    row['Comments'],
                    row['Favorite_count'],
                    row['Definition'],
                    row['Caption_status']
                   )



            cursor.execute(insert_query,values)
            mydb.commit()


def comments_table():
    mydb=psycopg2.connect(host="localhost",
                          user="postgres",
                          password=1999,
                          database="Youtubedata",
                          port="5432")
    cursor=mydb.cursor()

    drop_query='''drop table if exists comments'''
    cursor.execute(drop_query)
    mydb.commit()


    create_query='''create table if not exists comments(comment_Id varchar(10000) primary key,
                                                        video_Id varchar(5000),
                                                        Comment_Text text,
                                                        Comment_Author varchar(1500),
                                                        Comment_published timestamp
                                                        )'''



    cursor.execute(create_query)
    mydb.commit()


    com_list=[]
    db=client["Youtube_Data"]
    coll1=db["channel_details"]
    for com_data in coll1.find({},{"_id":0,"comment_information":1}):
        for i in range(len(com_data["comment_information"])):
            com_list.append(com_data["comment_information"][i])
    df3=pd.DataFrame(com_list)


    for index,row in df3.iterrows():
            insert_query='''insert into comments(comment_Id,
                                                 video_Id,
                                                 Comment_Text,
                                                 Comment_Author,
                                                 Comment_published
                                                )

                                                 values(%s,%s,%s,%s,%s)'''
            values=(row['comment_Id'],
                    row['video_Id'],
                    row['Comment_Text'],
                    row['Comment_Author'],
                    row['Comment_published']
                   )



            cursor.execute(insert_query,values)
            mydb.commit()



def tables():
    channels_table()
    playlist_table()
    videos_table()
    comments_table()
    
    return "Tables created successfully"


def show_channels_table():
    ch_list=[]
    db=client["Youtube_Data"]
    coll1=db["channel_details"]
    for ch_data in coll1.find({},{"_id":0,"channel_information":1}):
        ch_list.append(ch_data["channel_information"])
    df=st.dataframe(ch_list)
    
    return df


def show_playlist_table():
    pl_list=[]
    db=client["Youtube_Data"]
    coll1=db["channel_details"]
    for pl_data in coll1.find({},{"_id":0,"playlist_information":1}):
        for i in range(len(pl_data["playlist_information"])):
            pl_list.append(pl_data["playlist_information"][i])
    df1=st.dataframe(pl_list)
    
    return df1


def show_videos_table(): 
    vi_list=[]
    db=client["Youtube_Data"]
    coll1=db["channel_details"]
    for vi_data in coll1.find({},{"_id":0,"viedo_information":1}):
        for i in range(len(vi_data["viedo_information"])):
            vi_list.append(vi_data["viedo_information"][i])
    df2=st.dataframe(vi_list)
    
    return df2


def show_comments_table():
    com_list=[]
    db=client["Youtube_Data"]
    coll1=db["channel_details"]
    for com_data in coll1.find({},{"_id":0,"comment_information":1}):
        for i in range(len(com_data["comment_information"])):
            com_list.append(com_data["comment_information"][i])
    df3=st.dataframe(com_list)
    
    return df3



#streamlit 

with st.sidebar:
    st.title(":blue[YOUTUBE DATA HARVESTING]")
    st.header("Data is Anywhere and Everywhere")
    st.caption("Python Scripting")
    st.caption("Data Collection")
    st.caption("MongoDB")
    st.caption("API Intergration")
    st.caption("Data Management using MongoDB and SQL")
    
channel_id=st.text_input("Enter the channel ID")    

if st.button("Collect & Store data"):
    ch_ids=[]
    db=client["Youtube_Data"]
    coll1=db["channel_details"]
    for ch_data in coll1.find({},{"_id":0,"channel_information":1}):
        ch_ids.append(ch_data["channel_information"]["Channel_Id"])
        
    if channel_id in ch_ids:
        st.success("Channel Details of the given channel id already exists")
        
    else:
        insert=channel_details(channel_id)
        st.success(insert)
        
if st.button("Migrate to sql"):
    Table=tables()
    st.success(Table)
    
show_table=st.radio("SELECT THE TABLE FOR VIEW",("CHANNELS","PLAYLISTS","VIDEOS","COMMENTS"))

if show_table =="CHANNELS":
    show_channels_table()
    
elif show_table =="PLAYLISTS":    
    show_playlist_table() 
    
elif show_table =="VIDEOS":    
    show_videos_table()    

elif show_table =="COMMENTS":    
    show_comments_table()
    

#sql connecti0n

mydb=psycopg2.connect(host="localhost",
                      user="postgres",
                      password=1999,
                      database="Youtubedata",
                      port="5432")
cursor=mydb.cursor()

question=st.selectbox("Select your question",("1. What are the names of all the videos and their corresponding channels",
                                              "2. Which channels have the most number of videos, and how many videos dothey have",
                                              "3. What are the top 10 most viewed videos and their respective channels",
                                              "4. How many comments were made on each video, and what are their corresponding video names",
                                              "5. Which videos have the highest number of likes, and what are their corresponding channel names",
                                              "6. What is the total number of likes and dislikes for each video, and what are their corresponding video names",
                                              "7. What is the total number of views for each channel, and what are their corresponding channel names", 
                                              "8. What are the names of all the channels that have published videos in the year 2022",
                                              "9. What is the average duration of all videos in each channel, and what are their corresponding channel names", 
                                              "10. Which videos have the highest number of comments, and what are their corresponding channel names")) 

if question=="1. What are the names of all the videos and their corresponding channels":
    query1='''select title as videos,channel_name as channelname from videos'''
    cursor.execute(query1)
    mydb.commit()
    t1=cursor.fetchall()
    df=pd.DataFrame(t1,columns=["video title","channel name"])
    st.write(df)
    
    
elif question=="2. Which channels have the most number of videos, and how many videos dothey have":
    query2='''select channel_name as channelname,total_videos as no_videos from channels
                order by total_videos desc'''
    cursor.execute(query2)
    mydb.commit()
    t2=cursor.fetchall()
    df2=pd.DataFrame(t2,columns=["channel name","No of videos"])
    st.write(df2)
    
elif question=="3. What are the top 10 most viewed videos and their respective channels":
    query3='''select views as views,channel_name as channelname,title as videotitle from videos
                where views is not null order by views desc limit 10'''
    cursor.execute(query3)
    mydb.commit()
    t3=cursor.fetchall()
    df3=pd.DataFrame(t3,columns=["views","channel name","videotitle"])
    st.write(df3)
    
elif question=="4. How many comments were made on each video, and what are their corresponding video names":
    query4='''select comments as no_comments,title as videotitle from videos where comments is not null'''
    cursor.execute(query4)
    mydb.commit()
    t4=cursor.fetchall()
    df4=pd.DataFrame(t4,columns=["no comments","videotitle"])
    st.write(df4)
    
elif question=="5. Which videos have the highest number of likes, and what are their corresponding channel names":
    query5='''select title as videotitle,channel_name as channelname,Likes as likecount
                from videos where likes is not null order by Likes desc'''
    cursor.execute(query5)
    mydb.commit()
    t5=cursor.fetchall()
    df5=pd.DataFrame(t5,columns=["videotitle","channelname","likecount"])
    st.write(df5)   
    
elif question=="6. What is the total number of likes and dislikes for each video, and what are their corresponding video names":
    query6='''select Likes as likecount,title as videotitle from videos'''
    cursor.execute(query6)
    mydb.commit()
    t6=cursor.fetchall()
    df6=pd.DataFrame(t6,columns=["likecount","videotitle"])
    st.write(df6)
    
elif question=="7. What is the total number of views for each channel, and what are their corresponding channel names":
    query7='''select channel_name as channelname ,views as totalviews from channels'''
    cursor.execute(query7)
    mydb.commit()
    t7=cursor.fetchall()
    df7=pd.DataFrame(t7,columns=["likecount","videotitle"])
    st.write(df7)
    
elif question=="8. What are the names of all the channels that have published videos in the year 2022":
    query8='''select title as video_title,published_date as videorelease,channel_name as channelname from videos
            where extract(year from published_date)=2022'''
    cursor.execute(query8)
    mydb.commit()
    t8=cursor.fetchall()
    df8=pd.DataFrame(t8,columns=["videotitle","published_date","channelname"])
    st.write(df8)
    
elif question=="9. What is the average duration of all videos in each channel, and what are their corresponding channel names":
    query9='''select channel_name as channelname,AVG(duration) as averageduration from videos group by channel_name'''
    cursor.execute(query9)
    mydb.commit()
    t9=cursor.fetchall()
    df9=pd.DataFrame(t9,columns=["channelname","averageduration"])
    T9=[]
    for index,row in df9.iterrows():
        channel_title=row["channelname"]
        average_duration=row["averageduration"]
        average_duration_str=str(average_duration)
        T9.append(dict(channeltitle=channel_title,avgduration=average_duration_str))
        df1=pd.DataFrame(T9)
        st.write(df1)
        
elif question=="10. Which videos have the highest number of comments, and what are their corresponding channel names":
    query10='''select title as videotitle, channel_name as channelname, comments as comments from videos where comments is
                not null order by comments desc'''
    cursor.execute(query10)
    mydb.commit()
    t10=cursor.fetchall()
    df10=pd.DataFrame(t10,columns=["video title","channel name","comments"])
    st.write(df10)        







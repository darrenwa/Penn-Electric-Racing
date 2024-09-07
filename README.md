I wrote my code using Jupyter notebooks. For the code, run each block one by one. I ran everything in VSCode. Input files have not been included for the sake of space.


Task A:
min speed = 0 mph
avg speed = 23.96 mph
max speed = 49.68 mph
total energy consumed = 4.85 kwh

Notes on Task A:
I read the csv by splitting between the labels and the numerical data for the sake of simplicity. I then found and created separate dataframes for each relevant id. I then created a dataframe for power and integrated that. For speed, I averaged the speeds of each wheel. For the average speed, I initially averaged all the entries, but I realized that the time interval between the entries wasn't necessarily constant. I then decided to integrate to find the total distance and divided by time. In retrospect, I probably didn't need to create separate dataframes for every wheel, current, and voltage, but it was helpful for visualizing the data and examining for possible abnormal data entries.


Task B:
min speed = 0 mph
avg speed = 29.16 mph
max speed = 97.81 mph
total energy consumed = 4.63 kwh

Notes on Task B:
I had to do a good bit of research on converting bytes to floats, endianness, mantissas, etc. After some testing, I created a function for translating to floats. During data processing, I opted to transform the timestamps into seconds format just to make operations easier. When I was looking through the speeds, I noticed a negative value, so I decided to replace abnormal values with the average of the neighboring values. For the current and voltage datapoints, I noticed that they didn't match up at a significant number of times. To fill in these values, I used a simple linear approximation using the nearest valid value on either side. From then I just used the same process as in Task A.


Task C:

I spent a good amount of time looking into lossless encoding methods and when they would result in the most significant saving of space. I then scrolled through the csv and noticed that the third column often had stretches of repeated zeros, so I implemented run-length encoding for the third column. I also saw that the intervals of time were small, so I used delta encoding for the first column. My encoded file ended up being 123 MB. When I did the encoding I went one row at a time, but I think by going column by column I could have done more encoding (e.g. run-length encoding on the delta values in column one).

Using the above encoding methods sacrificed some readibility in the encoded file, but I decided that the file size decrease was significant enough to warrant the encoding. I believe my code to be fairly easy to debug, since you can specify indices for what to encode. While the encoded format isn't very human-readable, it is still fairly simple to verify that matches up with the original given a small sample of data. It is also easy to index through the encoded format, since each new line is marked with a |. In regards to portability/extensibility, I simply converted everything to a string, so endianness on different systems is less of an issue. To protect agaisnt corruption, some things I would consider implementing are checksums/Reed-Solomon error detection for data integrity. It would also be worth considering implementing range checks to catch unexpected values (e.g. negative speed). These would have to be implemented with consideration to file size, as something like Reed-Solomon requires additional data.


What I learned:

Doing data analysis definitely introduced me to some new Python libraries I wasn't too familiar with before. More importantly than code syntax, I learned a lot about the importance of data formatting and encoding. Before I really only associated data formats with file size, but now I know there are many more factors to consider. I'm really interested in working with embedded systems, which would be new to me, and how their resource constraints affect programming. 


What was the hardest part?

Doing research on various encoding methods and implementing them was the most difficult, but also the most fun. It was really cool seeing the file size decrease as I used more advanced encoding methods. It was tough to decide which route to go down, but I decided to stick with string encoding since it seemed the most accessible. I want to try some of the other techniques I saw, like huffman encoding or variable length integer encoding. 


What are some of the pros and cons of each log format? Why?

The csv-ish file format is a lot more readable before any processing. The older file format is more difficult to read due to the hexadecimal/byte format. The csv one also doesn't need to worry about endianness. The older file format is convenient with the grouping of relevant data together (e.g. the wheel speeds). Also, after a very brief comparison of file sizes, it seems that the csv format is more efficient at storing data than the older format (285 MB for > 16 million entries vs 41 MB for 1-2 million entries).


How fast are your parser(s)? What could you do to improve performance? What's the time complexity of your algorithm?

My parsers take about 10-20 seconds, with the vast majority of time coming from reading the csv file. I think I might be able to improve performance by doing a bit more preprocessing/skipping a few steps, but I don't see any major improvements that might be made. I believe the time complexity of each is O(n).


Can you create a graph of some of the data?

Yes, I used pandas' plot method to graph various data. A couple interesting notes: 
-For task B, the maximum/average speed for the right wheel velocity were higher than their counterparts for the left wheel velocity. My guess is this has something to do with turns or uneven force distribution.
-The power is in some cases negative. I was initially confused, but then I did some research and found out about regenerative braking.
-The speed reaches zero in the middle of the file for a span of time. After looking up the endurance race rules, I assume this is when the switching of drivers occurs.


Why might front and rear wheel speeds produce different results?

The aerodynamics and weight distribution may affect the wheels' traction, which could potentially affect wheel speed. The power delivery can also impact wheel speed, particualrly in the case of rear wheel drive. Other potential factors include the suspension design or wheels slipping. 


Each piece of data has to be sent out separately, meaning that you could get two different
pieces of information you need at different times. How did you deal with this? Is there a
better way?

As previously mentioned, I filled in missing values by approximating with a line between the nearest valid values. I think you might get a better approximation with higher order functions (spline interpolation), but this would be more difficult to compute and thus increase runtime. 


How fast are your encoders/decoders? How could you improve some of the performance?

My encoder/decoder each take around 40 seconds to run. Ignoring the reading of data, my encoder runs in ~15 seconds and my decoder in ~30 seconds. I think both of these (especially the decoder) could be improved if I used something other than strictly string encoding. My decoder relies heavily on indexing the string multiple times per row, which is inefficient. 
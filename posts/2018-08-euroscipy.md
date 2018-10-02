# NumPy presentations at EuroScipy 2018

*September 24, 2018*

https://bids.berkeley.edu/news/numpy-presentations-euroscipy-2018

I recently attended EuroScipy 2018, which was held in the beautiful city of Trento, Italy. The conference, attended by about 200 people from all over Europe, began with two days of tutorials, then two days of talks, and ended with a sprint day.

I gave a 90 minute tutorial on wrapping c-code in Python, attended by about 60 people. We wrote a Python program to create a mandlebrot image, then rewrote it in C and learned how to call the external c function from ctypes, cffi, cython, and cppyy. Along the way we compared the code complexity, speed, and maintainability of all four methods.

I also gave this 15 minute talk about the status of NumPy, and ended by encouraging the group to further their discussions and attend at the sprints.

I had many interesting conversations with developers of pandas, PyPy, scikit-learn as well as people from academia and industry. It deepened my understanding of the direction NumPy should take, especially around the refactoring of dtypes. Most people are quite happy with NumPy itself, but wished there were a standard set of tutorials and a gallery of usage examples, perhaps using "sphinx-gallery" as an infrastructure to show off NumPy features.

About 50 people attended the sprints, 25% of the conference attendees. After a two hour training session, we reached a peak of 6 people around the NumPy table. Two were veteran GitHub users, and they concentrated on working through new pull requests. The others expressed surprise at the difficulty of issues labeled 'easy' – labeling is  somewhat subjective, after all!  We spent some time triaging issues, closing 6 and opening 2 new ones.

The 'corridor track' was perhaps the most valuable part of the conference. The program was not overwhelming, so there was time at the conference to talk to people. Many people were on the social media chat app, as well, and we ended up spending many of the evenings together in local restaraunts and pubs.

My goal was to make contributing to the Open Source scientific Python stack, and NumPy in particular, more accessible. I feel I took a good step in the right direction.

-- Matti Picus

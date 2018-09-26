# Week of SciPy ↔ NumPy

Ralf Gommers: 23-29 July 2018
David Cournapeau: 23 July 2018

## Agenda

### Monday

- [NumPy Roadmap](https://github.com/numpy/numpy/wiki/NumPy-roadmap-v2); [v3 working document](https://hackmd.io/MxS4vff3TUGw29D2k_LIqA#)
- NumPy governance & grant governance
    - Blog post describing process at BIDS
        - History of application, how it came to be the way it is now (hiring of Matti & Tyler, etc.)
        - How are we aiming to spend the money? What kinds of activities do we support?
        - How do we set priorities? Evolution of decision making process & priorities.
          - Note differences between grant-related and technical / NumPy decisions
          - Trello board
          - Internal discussions of Trello cards
          - Roadmap
          - engage with the interests of community
          - increase code review activity in order to lighten the load on the volunteers; try to increase tractability of large PRs; use our time to focus specific questions for the core devs (both for our PRs and those from community)
          - "Innocuous" cleanups / refactoring while roadmap is being built
          - Monthly "Office Hours"?
      - Open questions
          - When we do have large code changes, how do we engage reviewers? Check ahead of time before large code changes?
          - How does NumFOCUS fit into this? [Ralf can write this section]
- Lunch w/ Jarrod, David C
    - Ways of getting more input/direction from the NumPy devs and community
    - How can we grow our community / make it more inclusive?
- What to do with `f2py`?
    - Ignore this issue for now. Refactoring will be work with little benefit.
- `np.linalg` vs `scipy.linalg`: official advice to users?  Room for overlap refactoring? Should we care about API consistency?
    - Simple rule for users: use SciPy if it is available, but don't add a dependency for functionality already available
    - OpenBLAS makes things better than before
    - API compatibility is a function-by-function decision
    - `numpy.dual` should probably be removed

### Tuesday

- SciPy paper (morning)
    - Sparse matrices tech improvements: internals have been rewritten, performance has improved a fair bit; perhaps Tyler should do a rough draft for, e.g., Hameer / CJ to refine?
    - Numerical optimization: probably a bit too long, but leave that editing until a later stage?
    - Community beyond the SciPy library: leave this section for now due to likely overlap with 'History' section being drafted by Stefan and Jarrod
    - Maintainers and contributors: Ralf will probably draft this
    - Impact now section: we should also mention LIGO (high profile project), shipped inside MacOS (credibility), something else from biology (that everyone has heard of?), Intel Python distribution?; pypinfo tool for collecting stats instead of using blog post citation (num downloads during 2017 for all Python versions: over 13 million or, more precisely, 13 096 468),  
    - Future development: perhaps this should reflect the key points of roadmap & roadmap section should just mention that we have one?
   
- Hike near Redwood Gate (afternoon, leave here around 1 and finish back by 4)

### Wednesday

- SciPy paper (morning)
   - delete most of stats PR content, keeping only the key distributions (multinomial, multivariate normal, and rv_histogram); keep the part on nan_policy design; then Ralf will refine after that 
- Stéfan away in a meeting until 10:30
- Onboarding [guides](http://www.numpy.org/devdocs/dev/gitwash/development_workflow.html) part 1 (after lunch)
- https://hackmd.io/yEHYN0bTT0SP4ssLcrtmpw#

### Thursday

- SciPy paper (morning)
- NumFOCUS: upcoming board elections & summit, pain points for projects
- Added some explanation about wrapping vs "grid-wrapping" to https://github.com/scipy/scipy/pull/206#issuecomment-408326019

### Friday

- Discuss [one obvious web portal to scientific Python](https://github.com/mrocklin/tutorials/issues/2#issuecomment-405113782) — note also PyOpenSci discussion
    - [Notes: Scientific Python landing page](https://hackmd.io/JL8ekLe8RA-wbTgC8ZKckg#)
- NEP 23 (backwards compatibility policy)
- SciPy paper

# domain-lang

A set of comain-specific language samples for simplifying common tasks.

My idea is that the world of programming gets a whole lot simpler when you use a series of targeted, special-purpose languages to define your system logic, and glue them together to build the more complex system.  There are a few things that make me think this might be true:
* General-purpose programming languages generally include a lot of 'boilerplate' to basically indicate what the file is about; a special purpose language could assume much of what is specified with boilerplate, avoiding the need to write it over and over again.
* A special-purpose language can typically describe a more complex idea more simply and accurately, and with lower likelihood of error.
* If special-purpose languages exclude chunks of functionality unneeded for their specific job, then it rules out the possibility that someone will have inappropriately placed that functionality in the poorly-suited module location.

There are already a lot of languages out there and I don't want to reinvent the wheel. Where a totally good DSL already exists, I'd like to use it, or at least take inspiration from it.

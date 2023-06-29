(code:Makefile_fft_filters)=
# `Makefile` for building `test_fft` and `test_filters`

## Source code
```{code-block} make 
:lineno-start: 1
:emphasize-lines: 0
# University of Florida EEL6528
# Tan F. Wong
# Jan 24, 2021

CXX = g++
CPPFLAGS = -O3 -DUSE_VOLK 
LDFLAGS = -lfftw3f -lfftw3f_threads -lvolk -luhd -lboost_program_options -lpthread 
OBJDIR := build

common_hdrs = fft.hpp filters.hpp

all: test_fft test_filters

test_fft: $(common_hdrs) $(OBJDIR) $(OBJDIR)/test_fft.o
	$(CXX) -o test_fft $(OBJDIR)/test_fft.o $(LDFLAGS)

test_filters: $(common_hdrs) $(OBJDIR) $(OBJDIR)/test_filters.o $(OBJDIR)/filters.o
	$(CXX) -o test_filters $(OBJDIR)/test_filters.o $(OBJDIR)/filters.o $(LDFLAGS)

$(OBJDIR):
	mkdir -p $@

$(OBJDIR)/%.o: %.cpp $(common_hdrs)
	$(CXX) $(CPPFLAGS) -c $< -o $@

clean: 
	rm -rf test_fft test_filters $(OBJDIR)
```

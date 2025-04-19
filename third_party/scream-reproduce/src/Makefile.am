AM_CPPFLAGS = $(CXX11_FLAGS) -I$(srcdir)/../util -I$(srcdir)/../scream_upstream/code
#AM_CXXFLAGS = $(PICKY_CXXFLAGS)

bin_PROGRAMS = ScreamClient ScreamServer

ScreamClient_SOURCES = \
	ScreamClient.cc Packet.hh Packet.cc Timerfd.hh Timerfd.cc \
	../scream_upstream/code/ScreamTx.h \
	../scream_upstream/code/ScreamTx.cpp \
	../scream_upstream/code/RtpQueue.h \
	../scream_upstream/code/RtpQueue.cpp \
	../scream_upstream/code/VideoEnc.h \
	../scream_upstream/code/VideoEnc.cpp
ScreamClient_LDADD = ../util/libutil.a

ScreamServer_SOURCES = \
	ScreamServer.cc Packet.hh Packet.cc Timerfd.hh Timerfd.cc \
	../scream_upstream/code/ScreamRx.h \
	../scream_upstream/code/ScreamRx.cpp
ScreamServer_LDADD = ../util/libutil.a

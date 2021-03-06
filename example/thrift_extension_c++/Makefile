BRPC_PATH = ../../
include $(BRPC_PATH)/config.mk
# Notes on the flags:
# 1. Added -fno-omit-frame-pointer: perf/tcmalloc-profiler use frame pointers by default
# 2. Added -D__const__= : Avoid over-optimizations of TLS variables by GCC>=4.8
CXXFLAGS = -std=c++0x -g -DENABLE_THRIFT_FRAMED_PROTOCOL -DDEBUG -D__const__= -pipe -W -Wall -Werror -Wno-unused-parameter -fPIC -fno-omit-frame-pointer
HDRS+=$(BRPC_PATH)/output/include
LIBS+=$(BRPC_PATH)/output/lib
HDRPATHS = $(addprefix -I, $(HDRS))
LIBPATHS = $(addprefix -L, $(LIBS))
COMMA=,
SOPATHS=$(addprefix -Wl$(COMMA)-rpath=, $(LIBS))

STATIC_LINKINGS += -lbrpc -lthrift -lgflags -Wl,--whole-archive -lbrpc_thrift -Wl,--no-whole-archive -levent

CLIENT_SOURCES = client.cpp
SERVER_SOURCES = server.cpp
PROTOS = $(wildcard *.proto)

PROTO_OBJS = $(PROTOS:.proto=.pb.o)
PROTO_GENS = $(PROTOS:.proto=.pb.h) $(PROTOS:.proto=.pb.cc)
CLIENT_OBJS = $(addsuffix .o, $(basename $(CLIENT_SOURCES))) 
SERVER_OBJS = $(addsuffix .o, $(basename $(SERVER_SOURCES))) 

.PHONY:all
all: echo_client echo_server native_server native_client libechothrift.a client.o server.o

.PHONY:clean
clean:
	@echo "Cleaning"
	@rm -rf echo_client echo_server $(PROTO_GENS) $(PROTO_OBJS) $(CLIENT_OBJS) $(SERVER_OBJS) native_server native_client EchoService.o echo_types.o libechothrift.a gen-cpp gen-py

echo_client:$(PROTO_OBJS) $(CLIENT_OBJS) libechothrift.a
	@echo "Linking $@"
	@$(CXX) $(LIBPATHS) $(SOPATHS) -Xlinker "-(" $^ -Xlinker "-)" $(STATIC_LINKINGS) $(DYNAMIC_LINKINGS) -o $@

echo_server:$(PROTO_OBJS) $(SERVER_OBJS) libechothrift.a
	@echo "Linking $@"
	@$(CXX) $(LIBPATHS) $(SOPATHS) -Xlinker "-(" $^ libechothrift.a -Xlinker "-)" $(STATIC_LINKINGS) $(DYNAMIC_LINKINGS) -o $@

%.o:%.cpp libechothrift.a
	@echo "Compiling $@"
	@$(CXX) -c $(HDRPATHS) $(CXXFLAGS) $< -o $@

%.o:%.cc libechothrift.a
	@echo "Compiling $@"
	@$(CXX) -c $(HDRPATHS) $(CXXFLAGS) $< -o $@

libechothrift.a:
	@echo "Generating thrift files"
	@thrift --gen cpp echo.thrift
	@thrift --gen py echo.thrift
	@$(CXX) -c gen-cpp/echo_types.cpp -o echo_types.o
	@$(CXX) -c gen-cpp/EchoService.cpp -o EchoService.o
	@ar -crv libechothrift.a EchoService.o echo_types.o

native_server: libechothrift.a
	@$(CXX) native_server.cpp gen-cpp/echo_types.cpp gen-cpp/EchoService.cpp $(HDRPATHS) $(LIBPATHS) $(CXXFLAGS) -lthriftnb -lthrift -levent -lpthread -lgflags -lbrpc -lbrpc_thrift -o native_server

native_client: libechothrift.a
	@$(CXX) native_client.cpp gen-cpp/echo_types.cpp gen-cpp/EchoService.cpp $(HDRPATHS) $(LIBPATHS) $(CXXFLAGS) -lthriftnb -lthrift -levent -lpthread -lgflags -lbrpc -lbrpc_thrift -o native_client

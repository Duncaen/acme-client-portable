include	Makefile.configure

PROG=		acme-client
SRCS=\
	acctproc.c\
	base64.c\
	certproc.c\
	chngproc.c\
	dbg.c\
	dnsproc.c\
	fileproc.c\
	http.c\
	jsmn.c\
	json.c\
	keyproc.c\
	main.c\
	netproc.c\
	parse.y\
	revokeproc.c\
	rsa.c\
	util.c\
	compats.c\

SRCS+=\
	sandbox-seccomp.c

OBJS=\
	$(patsubst %.c,%.o,$(filter %.c,${SRCS}))\
	$(patsubst %.y,%.o,$(filter %.y,${SRCS}))

MAN=		acme-client.1 acme-client.conf.5

LDADD=		-ltls -lssl -lcrypto -lresolv -lseccomp

CFLAGS+=	-W -Wall -Isrc/usr.sbin/acme-client
CFLAGS+=	-Wstrict-prototypes -Wmissing-prototypes
CFLAGS+=	-Wmissing-declarations
CFLAGS+=	-Wshadow -Wpointer-arith
CFLAGS+=	-Wsign-compare -Wunused
CFLAGS+=	-D_DEFAULT_SOURCE

all: $(PROG)

clean:
	rm -f $(PROG) $(OBJS)

distclean: clean
	rm -f config.h config.log Makefile.configure

.PHONY: all clean distclean

${SRCS}: config.h

${PROG}: ${OBJS}
	${CC} ${LDFLAGS} -o $@ $^ ${LDADD}

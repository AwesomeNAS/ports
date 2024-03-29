# Created by: Lukas Slebodnik <lukas.slebodnik@intrak.sk>
# $FreeBSD$

PORTNAME=	sssd
PORTVERSION=	1.11.7
PORTREVISION=	6
CATEGORIES=	security
MASTER_SITES=	https://fedorahosted.org/released/${PORTNAME}/ \
		http://mirrors.rit.edu/zi/

MAINTAINER=	lukas.slebodnik@intrak.sk
COMMENT=	System Security Services Daemon

LICENSE=	GPLv3+
LICENSE_FILE=	${WRKSRC}/COPYING

LIB_DEPENDS=	libpopt.so:devel/popt \
		libtalloc.so:devel/talloc \
		libtevent.so:devel/tevent \
		libtdb.so:databases/tdb \
		libldb.so:databases/ldb \
		libcares.so:dns/c-ares \
		libdbus-1.so:devel/dbus \
		libdhash.so:devel/ding-libs \
		libpcre.so:devel/pcre \
		libunistring.so:devel/libunistring \
		libnss3.so:security/nss \
		libsasl2.so:security/cyrus-sasl2 \
		libinotify.so:devel/libinotify
BUILD_DEPENDS=	xmlcatalog:textproc/libxml2 \
		docbook-xsl>=1:textproc/docbook-xsl \
		xsltproc:textproc/libxslt \
		xmlcatmgr:textproc/xmlcatmgr \
		krb5>=1.10:security/krb5 \
		nsupdate:dns/bind99

GNU_CONFIGURE=	yes
CONFIGURE_ARGS=	--with-selinux=no --with-semanage=no \
		--with-ldb-lib-dir=${LOCALBASE}/lib/shared-modules/ldb \
		--with-xml-catalog-path=${LOCALBASE}/share/xml/catalog \
		--with-libnl=no --with-init-dir=no --datadir=${DATADIR} \
		--docdir=${DOCSDIR} --with-pid-path=/var/run \
		--localstatedir=/var --enable-pammoddir=${PREFIX}/lib \
		--with-db-path=/var/db/sss --with-pipe-path=/var/run/sss \
		--with-pubconf-path=/var/run/sss --with-mcache-path=/var/db/sss_mc \
		--with-unicode-lib=libunistring --with-autofs=no \
		--disable-cifs-idmap-plugin --disable-config-lib
CFLAGS+=	-fstack-protector-all
PLIST_SUB=	PYTHON_VER=${PYTHON_VER}
#DEBUG_FLAGS=	-g
MAKE_ENV+=	LINGUAS="bg de eu es fr hu id it ja nb nl pl pt ru sv tg tr uk zh_CN zh_TW"
SUB_FILES=	pkg-message

USE_LDCONFIG=	yes
USE_OPENLDAP=	yes
USES=		autoreconf cpe gettext gmake iconv libtool pathfix pkgconfig \
		python:2 shebangfix gssapi:mit
INSTALL_TARGET=	install-strip
CPE_VENDOR=	fedoraproject

python_CMD=	${SETENV} python2
SHEBANG_FILES=	src/tools/sss_obfuscate \
		src/sbus/sbus_codegen

USE_RC_SUBR=	${PORTNAME}
PORTDATA=	*

OPTIONS_DEFINE=	DOCS SMB
OPTIONS_SUB=	yes

SMB_DESC=		Install IPA and AD providers (requires Samba4)
SMB_BUILD_DEPENDS=	samba42>=4.2.11:net/samba42
SMB_CONFIGURE_WITH=	samba

post-patch:
	@${REINPLACE_CMD} -e 's|SIGCLD|SIGCHLD|g' ${WRKSRC}/src/util/signal.c
	@${REINPLACE_CMD} -e 's|NSS_STATUS_NOTFOUND|NS_NOTFOUND|g' \
		-e 's|NSS_STATUS_UNAVAIL|NS_UNAVAIL|g' \
		-e 's|NSS_STATUS_TRYAGAIN|NS_TRYAGAIN|g' \
		-e '/ETIME/d' \
		-e 's|NSS_STATUS_SUCCESS|NS_SUCCESS|g' \
		${WRKSRC}/src/sss_client/common.c
	@${REINPLACE_CMD} \
		-e 's|pam_modutil_getlogin(pamh)|getlogin()|g' \
		${WRKSRC}/src/sss_client/pam_sss.c
	@${REINPLACE_CMD} \
		-e 's|install-data-hook install-dist_initSCRIPTS|install-dist_initSCRIPTS|g' \
		-e 's|install-data-hook|notinstall-data-hook|g' \
		${WRKSRC}/Makefile.am
	@${REINPLACE_CMD} -e 's|/etc/sssd/|${ETCDIR}/|g' \
		-e 's|/etc/openldap/|${LOCALBASE}/etc/openldap/|g' \
		${WRKSRC}/src/man/*xml
	@${CP} ${FILESDIR}/bsdnss.c ${WRKSRC}/src/sss_client/bsdnss.c
	@${CP} ${FILESDIR}/sss_bsd_errno.h ${WRKSRC}/src/util/sss_bsd_errno.h

post-install:
	${INSTALL_DATA} ${WRKSRC}/src/examples/sssd-example.conf \
		${STAGEDIR}${ETCDIR}/sssd.conf.sample
	${LN} -sf nss_sss.so ${STAGEDIR}${PREFIX}/lib/nss_sss.so.1
# clean these up from the install; we create them in rc script start_precmd
.for d in db/sss db/sss_mc log/sssd run/sss/krb5.include.d run/sss/private run/sss
	@${RMDIR} ${STAGEDIR}/var/${d}
.endfor
# clean unused man dirs
.for i in nl/man1 nl/man5 pt/man1 pt/man5
	@${RMDIR} ${STAGEDIR}${PREFIX}/man/${i}
.endfor

.include <bsd.port.mk>

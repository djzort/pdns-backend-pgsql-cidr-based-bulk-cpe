--
-- PostgreSQL database dump
--

SET statement_timeout = 0;
SET client_encoding = 'UTF8';
SET standard_conforming_strings = on;
SET check_function_bodies = false;
SET client_min_messages = warning;

--
-- Name: plpgsql; Type: EXTENSION; Schema: -; Owner:
--

CREATE EXTENSION IF NOT EXISTS plpgsql WITH SCHEMA pg_catalog;


--
-- Name: EXTENSION plpgsql; Type: COMMENT; Schema: -; Owner:
--

COMMENT ON EXTENSION plpgsql IS 'PL/pgSQL procedural language';


--
-- Name: ip4r; Type: EXTENSION; Schema: -; Owner:
--

CREATE EXTENSION IF NOT EXISTS ip4r WITH SCHEMA public;


SET search_path = public, pg_catalog;

--
-- Name: any_id_query(text, integer); Type: FUNCTION; Schema: public; Owner: postgres
--

CREATE FUNCTION any_id_query(qtext text, qnumber integer, OUT content text, OUT ttl integer, OUT prio integer, OUT type text, OUT domain_id integer, OUT disabled boolean, OUT name text, OUT auth boolean) RETURNS SETOF record
    LANGUAGE plpgsql STABLE ROWS 1
    AS $_$
DECLARE
  ip integer[];
BEGIN

ip := REGEXP_MATCHES(qtext, '^(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.in-addr\.arpa$');

IF ip IS NOT NULL THEN

RETURN QUERY EXECUTE format('SELECT
  format(templ, %2$s, %3$s, %4$s, %5$s),
  ttl, 0, type::text, domain_id, false, name, true
FROM (
  SELECT *,
    ''%1$s''::text AS name
  FROM cpe_pseudo_records
  WHERE
    range >> ''%2$s.%3$s.%4$s.%5$s''::ipaddress
    AND domain_id = %6$s
)
AS foo;', qtext, ip[4], ip[3], ip[2], ip[1], qnumber);

ELSE

ip := REGEXP_MATCHES(qtext, '(\d{1,3})\D(\d{1,3})\D(\d{1,3})\D(\d{1,3})');

IF ip IS NOT NULL THEN

RETURN QUERY EXECUTE format('SELECT
  ''%2$s.%3$s.%4$s.%5$s''::text,
  ttl, 0, type::text, domain_id, false, name, true
FROM (
  SELECT *,
    ''%1$s''::text AS name
  FROM cpe_pseudo_records
  WHERE
    range >> ''%2$s.%3$s.%4$s.%5$s''::ipaddress
    AND domain_id = %6$s
)
AS foo
WHERE format(templ, %2$s, %3$s, %4$s, %5$s) = ''%1$s''
;', qtext, ip[1], ip[2], ip[3], ip[4], qnumber);


END IF;

END IF;

RETURN;

END
$_$;


ALTER FUNCTION public.any_id_query(qtext text, qnumber integer, OUT content text, OUT ttl integer, OUT prio integer, OUT type text, OUT domain_id integer, OUT disabled boolean, OUT name text, OUT auth boolean) OWNER TO postgres;

--
-- Name: any_query(text); Type: FUNCTION; Schema: public; Owner: postgres
--

CREATE FUNCTION any_query(qtext text, OUT content text, OUT ttl integer, OUT prio integer, OUT type text, OUT domain_id integer, OUT disabled boolean, OUT name text, OUT auth boolean) RETURNS SETOF record
    LANGUAGE plpgsql STABLE ROWS 1
    AS $_$
DECLARE
  ip integer[];
BEGIN

ip := REGEXP_MATCHES(qtext, '^(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.in-addr\.arpa$');

IF ip IS NOT NULL THEN

RETURN QUERY EXECUTE format('SELECT
  concat(format(templ, %2$s, %3$s, %4$s, %5$s),''.'',basedomain),
  ttl, 0, ''PTR''::text, h_int(basedomain), false, ''%1$s''::text, true
  FROM cpe_ranges INNER JOIN cpe_formats ON (cpe_formats.id = cpe_ranges.format_id)
  WHERE
    range >> ''%2$s.%3$s.%4$s.%5$s''::ipaddress
;', qtext, ip[4], ip[3], ip[2], ip[1]);

ELSE

ip := REGEXP_MATCHES(qtext, '(\d{1,3})\D(\d{1,3})\D(\d{1,3})\D(\d{1,3})');

IF ip IS NOT NULL THEN

RETURN QUERY EXECUTE format('SELECT
  ''%2$s.%3$s.%4$s.%5$s''::text,
  ttl, 0, ''A''::text, h_int(basedomain), false, ''%1$s''::text, true
  FROM cpe_ranges INNER JOIN cpe_formats ON (cpe_formats.id = cpe_ranges.format_id)
  WHERE 1=1
    AND range >> ''%2$s.%3$s.%4$s.%5$s''::ipaddress
    AND concat(format(templ, %2$s, %3$s, %4$s, %5$s),''.'',basedomain) = ''%1$s''
;', qtext, ip[1], ip[2], ip[3], ip[4]);


END IF;

END IF;

RETURN;

END
$_$;


ALTER FUNCTION public.any_query(qtext text, OUT content text, OUT ttl integer, OUT prio integer, OUT type text, OUT domain_id integer, OUT disabled boolean, OUT name text, OUT auth boolean) OWNER TO postgres;

--
-- Name: basic_query(text, text); Type: FUNCTION; Schema: public; Owner: postgres
--

CREATE FUNCTION basic_query(qtype text, qtext text, OUT content text, OUT ttl integer, OUT prio integer, OUT type text, OUT domain_id integer, OUT disabled boolean, OUT name text, OUT auth boolean) RETURNS SETOF record
    LANGUAGE plpgsql STABLE ROWS 1
    AS $_$
DECLARE
  ip integer[];
BEGIN

  CASE qtype
    WHEN 'SOA', 'NS' THEN

        ip := REGEXP_MATCHES(qtext, '^(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.in-addr\.arpa$');

        IF ip IS NOT NULL THEN

            RETURN QUERY EXECUTE format('
SELECT
content::text,cpe_authorities.ttl,prio,type::text,h_int(''%1$s''::text),false,''%1$s''::text,true
FROM cpe_authorities,cpe_ranges
WHERE 1=1
AND range >> ''%5$s.%4$s.%3$s.0''::ipaddress
AND type = ''%2$s''
', qtext, qtype, ip[1], ip[2], ip[3]);

        ELSE

            ip := REGEXP_MATCHES(qtext, '^(\d{1,3})\.(\d{1,3})\.in-addr\.arpa$');

            IF ip IS NOT NULL THEN

                RETURN QUERY EXECUTE format('
SELECT
content::text,cpe_authorities.ttl,prio,type::text,h_int(''%1$s''::text),false,''%1$s''::text,true
FROM cpe_authorities,cpe_ranges
WHERE 1=1
AND range >> ''%4$s.%3$s.0.0''::ipaddress
AND type = ''%2$s''
', qtext, qtype, ip[1], ip[2]);


            ELSE

                RETURN QUERY EXECUTE format('
SELECT
content::text,cpe_authorities.ttl,prio,type::text,h_int(''%1$s''::text),false,basedomain::text,true
FROM cpe_authorities,cpe_formats
WHERE 1=1
AND basedomain = ''%1$s''
AND type = ''%2$s''
', qtext, qtype);


            END IF;

        END IF;

  ELSE

RETURN;

  END CASE;

END
$_$;


ALTER FUNCTION public.basic_query(qtype text, qtext text, OUT content text, OUT ttl integer, OUT prio integer, OUT type text, OUT domain_id integer, OUT disabled boolean, OUT name text, OUT auth boolean) OWNER TO postgres;

--
-- Name: disolve_cidr(cidr, integer); Type: FUNCTION; Schema: public; Owner: postgres
--

CREATE FUNCTION disolve_cidr(net cidr, maximum integer) RETURNS SETOF cidr
    LANGUAGE plpgsql STABLE
    AS $$
DECLARE
  r cidr;
BEGIN

  IF masklen(net) = maximum THEN
    RETURN NEXT net;
    RETURN;
  END IF;

  FOR r IN SELECT * FROM disolve_cidr( set_masklen(net,masklen(net)+1)::cidr, maximum )
    LOOP RETURN NEXT r;
    END LOOP;

  FOR r IN SELECT * FROM disolve_cidr( set_masklen(broadcast(net),masklen(net)+1)::cidr, maximum )
    LOOP RETURN NEXT r;
    END LOOP;

  RETURN;

END $$;


ALTER FUNCTION public.disolve_cidr(net cidr, maximum integer) OWNER TO postgres;

--
-- Name: fake_inaddr_arpa(cidr); Type: FUNCTION; Schema: public; Owner: postgres
--

CREATE FUNCTION fake_inaddr_arpa(net cidr) RETURNS SETOF text
    LANGUAGE plpgsql STABLE
    AS $_$
DECLARE
  r text;
  ip integer[];
BEGIN

  IF masklen(net) = 16 THEN
    ip := REGEXP_MATCHES(host(net)::text, '^(\d{1,3})\.(\d{1,3})\.0\.0$');
    RETURN NEXT format('%s.%s.in-addr.arpa',ip[2],ip[1]);
    RETURN;
  END IF;

  IF masklen(net) = 24 THEN
    ip := REGEXP_MATCHES(host(net)::text, '^(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.0$');
    RETURN NEXT format('%s.%s.%s.in-addr.arpa',ip[3],ip[2],ip[1]);
    RETURN;
  END IF;

  FOR r IN SELECT * FROM fake_inaddr_arpa( set_masklen(net,masklen(net)+1)::cidr )
    LOOP RETURN NEXT r;
    END LOOP;

  FOR r IN SELECT * FROM fake_inaddr_arpa( set_masklen(broadcast(net),masklen(net)+1)::cidr )
    LOOP RETURN NEXT r;
    END LOOP;

  RETURN;

END $_$;


ALTER FUNCTION public.fake_inaddr_arpa(net cidr) OWNER TO postgres;

--
-- Name: get_domain_metadata_query(text, text); Type: FUNCTION; Schema: public; Owner: postgres
--

CREATE FUNCTION get_domain_metadata_query(qname text, qkind text, OUT content text) RETURNS SETOF text
    LANGUAGE plpgsql STABLE ROWS 1
    AS $$
BEGIN

RETURN;

END
$$;


ALTER FUNCTION public.get_domain_metadata_query(qname text, qkind text, OUT content text) OWNER TO postgres;

--
-- Name: h_int(text); Type: FUNCTION; Schema: public; Owner: postgres
--

CREATE FUNCTION h_int(text) RETURNS integer
    LANGUAGE sql
    AS $_$
 select ('x'||substr(md5($1),1,8))::bit(32)::int;
$_$;


ALTER FUNCTION public.h_int(text) OWNER TO postgres;

--
-- Name: id_query(text, text, integer); Type: FUNCTION; Schema: public; Owner: postgres
--

CREATE FUNCTION id_query(qtype text, qtext text, qnumber integer, OUT content text, OUT ttl integer, OUT prio integer, OUT type text, OUT domain_id integer, OUT disabled boolean, OUT name text, OUT auth boolean) RETURNS SETOF record
    LANGUAGE plpgsql STABLE ROWS 1
    AS $_$
DECLARE
  ip integer[];
BEGIN

  CASE qtype
    WHEN 'SOA', 'NS' THEN

RETURN QUERY EXECUTE format('SELECT
content::text,ttl,prio,type::text,id,false,''%2$s''::text,true FROM cpe_authorities
WHERE type = ''%1$s''
', qtype, qtext);

    WHEN 'A' THEN

ip := REGEXP_MATCHES(qtext, '^(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.in-addr\.arpa$');

IF ip IS NOT NULL THEN

RETURN QUERY EXECUTE format('SELECT
  name,
  ttl,
  0,
  type::text,
  domain_id,
  false,
  format(templ, %3$s, %4$s, %5$s, %6$s) AS content,
  true
FROM (
  SELECT *,
    ''%2$s''::text AS name
  FROM cpe_pseudo_records
  WHERE
    range >> ''%3$s.%4$s.%5$s.%6$s''::ipaddress AND
    type = ''%1$s'' AND
    domain_id = %7$s
)
AS foo;', qtype, qtext, ip[4], ip[3], ip[2], ip[1], qnumber);

ELSE

  RETURN;

END IF;

END CASE;

END
$_$;


ALTER FUNCTION public.id_query(qtype text, qtext text, qnumber integer, OUT content text, OUT ttl integer, OUT prio integer, OUT type text, OUT domain_id integer, OUT disabled boolean, OUT name text, OUT auth boolean) OWNER TO postgres;

--
-- Name: info_all_master_query(); Type: FUNCTION; Schema: public; Owner: postgres
--

CREATE FUNCTION info_all_master_query(OUT id integer, OUT name text, OUT master text, OUT last_check integer, OUT notified_serial integer, OUT type text) RETURNS SETOF record
    LANGUAGE plpgsql STABLE ROWS 1
    AS $$
DECLARE
  r iprange;
BEGIN

RETURN QUERY EXECUTE '
SELECT
h_int(basedomain::text),
basedomain::text,
''''::text,
0,
to_char(now(),''YYYYMMDDHH24'')::integer,
''MASTER''::text
FROM cpe_formats
';

FOR r IN SELECT cpe_ranges.range FROM cpe_ranges LOOP

RETURN QUERY EXECUTE format('
SELECT
h_int(fake_inaddr_arpa),
fake_inaddr_arpa,
''''::text,
0,
to_char(now(),''YYYYMMDDHH24'')::integer,
''MASTER''::text
FROM fake_inaddr_arpa(''%s'');
',r);

END LOOP;

END
$$;


ALTER FUNCTION public.info_all_master_query(OUT id integer, OUT name text, OUT master text, OUT last_check integer, OUT notified_serial integer, OUT type text) OWNER TO postgres;

SET default_tablespace = '';

SET default_with_oids = false;

--
-- Name: cpe_authorities; Type: TABLE; Schema: public; Owner: postgres; Tablespace:
--

CREATE TABLE cpe_authorities (
    id integer NOT NULL,
    type character varying(10) DEFAULT NULL::character varying,
    content character varying(65535) DEFAULT NULL::character varying,
    ttl integer,
    prio integer
);


ALTER TABLE public.cpe_authorities OWNER TO postgres;

--
-- Name: cpe_authorities_id_seq; Type: SEQUENCE; Schema: public; Owner: postgres
--

CREATE SEQUENCE cpe_authorities_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;


ALTER TABLE public.cpe_authorities_id_seq OWNER TO postgres;

--
-- Name: cpe_authorities_id_seq; Type: SEQUENCE OWNED BY; Schema: public; Owner: postgres
--

ALTER SEQUENCE cpe_authorities_id_seq OWNED BY cpe_authorities.id;


--
-- Name: cpe_domainmetadata; Type: TABLE; Schema: public; Owner: postgres; Tablespace:
--

CREATE TABLE cpe_domainmetadata (
    id integer NOT NULL,
    domain_id integer,
    kind character varying(32),
    content text
);


ALTER TABLE public.cpe_domainmetadata OWNER TO postgres;

--
-- Name: cpe_formats; Type: TABLE; Schema: public; Owner: postgres; Tablespace:
--

CREATE TABLE cpe_formats (
    id integer NOT NULL,
    created timestamp with time zone,
    ttl integer NOT NULL,
    basedomain character varying(65535) NOT NULL,
    templ character varying(255) NOT NULL,
    notified_serial integer DEFAULT 0 NOT NULL,
    CONSTRAINT c_lowercase_cpe_formats_basedomain CHECK (((basedomain)::text = lower((basedomain)::text))),
    CONSTRAINT c_lowercase_cpe_formats_templ CHECK (((templ)::text = lower((templ)::text)))
);


ALTER TABLE public.cpe_formats OWNER TO postgres;

--
-- Name: cpe_formats_id_seq; Type: SEQUENCE; Schema: public; Owner: postgres
--

CREATE SEQUENCE cpe_formats_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;


ALTER TABLE public.cpe_formats_id_seq OWNER TO postgres;

--
-- Name: cpe_formats_id_seq; Type: SEQUENCE OWNED BY; Schema: public; Owner: postgres
--

ALTER SEQUENCE cpe_formats_id_seq OWNED BY cpe_formats.id;


--
-- Name: cpe_ranges; Type: TABLE; Schema: public; Owner: postgres; Tablespace:
--

CREATE TABLE cpe_ranges (
    id bigint NOT NULL,
    format_id integer NOT NULL,
    range iprange NOT NULL,
    CONSTRAINT c_netmask_cpe_ranges CHECK ((masklen(range) < 25))
);


ALTER TABLE public.cpe_ranges OWNER TO postgres;

--
-- Name: cpe_ranges_id_seq; Type: SEQUENCE; Schema: public; Owner: postgres
--

CREATE SEQUENCE cpe_ranges_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;


ALTER TABLE public.cpe_ranges_id_seq OWNER TO postgres;

--
-- Name: cpe_ranges_id_seq; Type: SEQUENCE OWNED BY; Schema: public; Owner: postgres
--

ALTER SEQUENCE cpe_ranges_id_seq OWNED BY cpe_ranges.id;


--
-- Name: domainmetadata_id_seq; Type: SEQUENCE; Schema: public; Owner: postgres
--

CREATE SEQUENCE domainmetadata_id_seq
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;


ALTER TABLE public.domainmetadata_id_seq OWNER TO postgres;

--
-- Name: domainmetadata_id_seq; Type: SEQUENCE OWNED BY; Schema: public; Owner: postgres
--

ALTER SEQUENCE domainmetadata_id_seq OWNED BY cpe_domainmetadata.id;


--
-- Name: id; Type: DEFAULT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY cpe_authorities ALTER COLUMN id SET DEFAULT nextval('cpe_authorities_id_seq'::regclass);


--
-- Name: id; Type: DEFAULT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY cpe_domainmetadata ALTER COLUMN id SET DEFAULT nextval('domainmetadata_id_seq'::regclass);


--
-- Name: id; Type: DEFAULT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY cpe_formats ALTER COLUMN id SET DEFAULT nextval('cpe_formats_id_seq'::regclass);


--
-- Name: id; Type: DEFAULT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY cpe_ranges ALTER COLUMN id SET DEFAULT nextval('cpe_ranges_id_seq'::regclass);


--
-- Data for Name: cpe_authorities; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY cpe_authorities (id, type, content, ttl, prio) FROM stdin;
2	NS	localhost	3360	0
1	SOA	localhost dnsadmin.staff.acme.com 2015031203 1200 180 1209600 3600	3360	0
\.


--
-- Name: cpe_authorities_id_seq; Type: SEQUENCE SET; Schema: public; Owner: postgres
--

SELECT pg_catalog.setval('cpe_authorities_id_seq', 3, true);


--
-- Data for Name: cpe_domainmetadata; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY cpe_domainmetadata (id, domain_id, kind, content) FROM stdin;
\.


--
-- Data for Name: cpe_formats; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY cpe_formats (id, created, ttl, basedomain, templ, notified_serial) FROM stdin;
2	\N	3360	cnat.acme.com	cnat-%1$s-%2$s-%3$s-%4$s	0
8	\N	3360	benchmarks.acme.com	bench-%1$s-%2$s-%3$s-%4$s	0
9	\N	3360	rfc1918.acme.com	rfc-%1$s-%2$s-%3$s-%4$s	0
1	\N	3360	test1.acme.com	cpe-%1$s-%2$s-%3$s-%4$s	0
\.


--
-- Name: cpe_formats_id_seq; Type: SEQUENCE SET; Schema: public; Owner: postgres
--

SELECT pg_catalog.setval('cpe_formats_id_seq', 9, true);


--
-- Data for Name: cpe_ranges; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY cpe_ranges (id, format_id, range) FROM stdin;
1	1	192.0.2.0/24
2	8	198.18.0.0/15
6	9	192.168.0.0/16
13	9	10.0.0.0/8
14	9	172.16.0.0/12
15	2	100.64.0.0/10
\.


--
-- Name: cpe_ranges_id_seq; Type: SEQUENCE SET; Schema: public; Owner: postgres
--

SELECT pg_catalog.setval('cpe_ranges_id_seq', 15, true);


--
-- Name: domainmetadata_id_seq; Type: SEQUENCE SET; Schema: public; Owner: postgres
--

SELECT pg_catalog.setval('domainmetadata_id_seq', 1, false);


--
-- Name: cpe_authorities_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres; Tablespace:
--

ALTER TABLE ONLY cpe_authorities
    ADD CONSTRAINT cpe_authorities_pkey PRIMARY KEY (id);


--
-- Name: cpe_formats_pk; Type: CONSTRAINT; Schema: public; Owner: postgres; Tablespace:
--

ALTER TABLE ONLY cpe_formats
    ADD CONSTRAINT cpe_formats_pk PRIMARY KEY (id);


--
-- Name: cpe_ranges_pk; Type: CONSTRAINT; Schema: public; Owner: postgres; Tablespace:
--

ALTER TABLE ONLY cpe_ranges
    ADD CONSTRAINT cpe_ranges_pk PRIMARY KEY (id);


--
-- Name: domainmetadata_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres; Tablespace:
--

ALTER TABLE ONLY cpe_domainmetadata
    ADD CONSTRAINT domainmetadata_pkey PRIMARY KEY (id);


--
-- Name: domainidmetaindex; Type: INDEX; Schema: public; Owner: postgres; Tablespace:
--

CREATE INDEX domainidmetaindex ON cpe_domainmetadata USING btree (domain_id);


--
-- Name: cpe_ranges_format_id_fkey; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY cpe_ranges
    ADD CONSTRAINT cpe_ranges_format_id_fkey FOREIGN KEY (format_id) REFERENCES cpe_formats(id) ON DELETE CASCADE;


--
-- Name: public; Type: ACL; Schema: -; Owner: postgres
--

REVOKE ALL ON SCHEMA public FROM PUBLIC;
REVOKE ALL ON SCHEMA public FROM postgres;
GRANT ALL ON SCHEMA public TO postgres;
GRANT ALL ON SCHEMA public TO PUBLIC;
GRANT USAGE ON SCHEMA public TO pdns;


--
-- Name: cpe_authorities; Type: ACL; Schema: public; Owner: postgres
--

REVOKE ALL ON TABLE cpe_authorities FROM PUBLIC;
REVOKE ALL ON TABLE cpe_authorities FROM postgres;
GRANT ALL ON TABLE cpe_authorities TO postgres;
GRANT SELECT ON TABLE cpe_authorities TO pdns;


--
-- Name: cpe_domainmetadata; Type: ACL; Schema: public; Owner: postgres
--

REVOKE ALL ON TABLE cpe_domainmetadata FROM PUBLIC;
REVOKE ALL ON TABLE cpe_domainmetadata FROM postgres;
GRANT ALL ON TABLE cpe_domainmetadata TO postgres;
GRANT SELECT ON TABLE cpe_domainmetadata TO pdns;


--
-- Name: cpe_formats; Type: ACL; Schema: public; Owner: postgres
--

REVOKE ALL ON TABLE cpe_formats FROM PUBLIC;
REVOKE ALL ON TABLE cpe_formats FROM postgres;
GRANT ALL ON TABLE cpe_formats TO postgres;
GRANT SELECT,UPDATE ON TABLE cpe_formats TO pdns;


--
-- Name: cpe_ranges; Type: ACL; Schema: public; Owner: postgres
--

REVOKE ALL ON TABLE cpe_ranges FROM PUBLIC;
REVOKE ALL ON TABLE cpe_ranges FROM postgres;
GRANT ALL ON TABLE cpe_ranges TO postgres;
GRANT SELECT ON TABLE cpe_ranges TO pdns;


--
-- PostgreSQL database dump complete
--


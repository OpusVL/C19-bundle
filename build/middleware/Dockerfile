FROM perl:5.30

RUN cpanm --force \
    Cookie::Baker \
    Data::Dumper \
    Data::UUID \
    DateTime \
    File::Slurp \
    HTTP::Cookies \
    HTTP::Request::Common \
    HTTP::Status \
    JSON::MaybeXS \
    POE \
    POE::Component::Client::HTTP \
    POE::Component::Client::Keepalive \
    POE::Component::Server::SimpleHTTP \
    Storable Data::Search \
    Test::POE::Client::TCP \
    Try::Tiny \
    URI \
    URI::QueryParam \
    XML::TreeBuilder

COPY ./C19-Prototype-Middleware /opt/C19

WORKDIR /opt/C19

EXPOSE 18080

CMD [ "perl", "./server.pl" ]


FROM ubuntu:16.04
MAINTAINER Brian Haas (bhaas@broadinstitute.org)
# with many thanks to Nicolas Delhomme (nicolas.delhomme@slu.se) and Bastian Schiffthaler (bastian.schiffthaler@umu.se)


RUN apt-get update && apt-get install -y build-essential curl unzip apache2 supervisor \
     default-jre wget git ftp nano python2.7-dev python-pip openssl openssh-server \
     vim emacs perl gcc g++ perl python automake make \
	wget git curl libdb-dev \
    zlib1g-dev bzip2 libncurses5-dev \
    texlive-latex-base \
    default-jre \
    python-pip python-dev \
        build-essential wget libghc-zlib-dev libncurses-dev libbz2-dev liblzma-dev libpcre3-dev libxml2-dev \
        libblas-dev gfortran git unzip ftp libzmq3-dev nano ftp fort77 libreadline-dev libcurl4-openssl-dev libx11-dev libxt-dev \
        x11-common libcairo2-dev libpng12-dev libreadline6-dev libjpeg8-dev pkg-config && \
        apt-get clean



RUN curl -L https://cpanmin.us | perl - App::cpanminus

RUN cpanm install DBD::SQLite
RUN cpanm install XML::Simple
RUN cpanm install CGI
RUN cpanm install HTML::Template
RUN cpanm install DB_File
RUN cpanm install URI::Escape




###################################################################################
## From base-r (https://github.com/rocker-org/rocker/blob/master/r-base/Dockerfile)

RUN apt-get update \ 
	&& apt-get install -y --no-install-recommends \
		ed \
		less \
		locales \
		vim-tiny \
		wget \
		ca-certificates \
		fonts-texgyre \
	&& rm -rf /var/lib/apt/lists/*

## Configure default locale, see https://github.com/rocker-org/rocker/issues/19
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
	&& locale-gen en_US.utf8 \
	&& /usr/sbin/update-locale LANG=en_US.UTF-8

ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8

## Now install R and littler, and create a link for littler in /usr/local/bin
## Also set a default CRAN repo, and make sure littler knows about it too
RUN apt-get update \
	&& apt-get install -y --no-install-recommends \
		littler \
        r-cran-littler \
		r-base \
		r-base-dev \
		r-recommended \
        && echo 'options(repos = c(CRAN = "https://cran.rstudio.com/"), download.file.method = "libcurl")' >> /etc/R/Rprofile.site \
        && echo 'source("/etc/R/Rprofile.site")' >> /etc/littler.r \
	&& ln -s /usr/share/doc/littler/examples/install.r /usr/local/bin/install.r \
	&& ln -s /usr/share/doc/littler/examples/install2.r /usr/local/bin/install2.r \
	&& ln -s /usr/share/doc/littler/examples/installGithub.r /usr/local/bin/installGithub.r \
	&& ln -s /usr/share/doc/littler/examples/testInstalled.r /usr/local/bin/testInstalled.r \
	&& install.r docopt \
	&& rm -rf /tmp/downloaded_packages/ /tmp/*.rds \
	&& rm -rf /var/lib/apt/lists/*



###################################
## Now on to rest of workshop setup



RUN groupadd -g 2000 training && useradd -m -u 2000 -g 2000 training
RUN echo 'training:training' | chpasswd
RUN chsh -s /bin/bash training
ENV HOME=/home/training
RUN echo "alias ll='ls -la -G'" >> /home/training/.profile
RUN usermod -G training,www-data training


WORKDIR /usr/local/src


#########
### GateOne SSH interface
#########
RUN git clone https://github.com/liftoff/GateOne/ && \
    cd GateOne && python setup.py install && \
        pip install tornado==4.5.3 && \
            python run_gateone.py --configure




# ports
EXPOSE 22 80 443 8787


# init

ADD supervisord.conf /etc/supervisor/conf.d/supervisord.conf

CMD ["/usr/bin/supervisord","-c","/etc/supervisor/conf.d/supervisord.conf"]


## install bioconductor packages:

RUN Rscript -e 'source("http://bioconductor.org/biocLite.R");library(BiocInstaller); biocLite("edgeR", dep = TRUE)'

###########
## Trinity



## set up tool config and deployment area:

ENV SRC /usr/local/src
ENV BIN /usr/local/bin

ENV TRINITY_VERSION="2.6.6"

WORKDIR $SRC

RUN TRINITY_URL="https://github.com/trinityrnaseq/trinityrnaseq/archive/Trinity-v${TRINITY_VERSION}.tar.gz" && \
    wget $TRINITY_URL

RUN tar -xvf Trinity-v${TRINITY_VERSION}.tar.gz

RUN cd trinityrnaseq-Trinity-v${TRINITY_VERSION} && make

ENV TRINITY_HOME $SRC/trinityrnaseq-Trinity-v${TRINITY_VERSION}


ENV PATH=${TRINITY_HOME}:${PATH}


## samtools
RUN wget https://github.com/samtools/samtools/releases/download/1.7/samtools-1.7.tar.bz2 && \
    tar xvf samtools-1.7.tar.bz2 && \
        cd samtools-1.7/ && \
            ./configure && make && make install

## Jellyfish
RUN wget https://github.com/gmarcais/Jellyfish/releases/download/v2.2.7/jellyfish-2.2.7.tar.gz && \
    tar xvf jellyfish-2.2.7.tar.gz && \
        cd jellyfish-2.2.7/ && \
            ./configure && make && make install


## Salmon
WORKDIR $SRC
RUN wget https://github.com/COMBINE-lab/salmon/releases/download/v0.9.1/Salmon-0.9.1_linux_x86_64.tar.gz && \
    tar xvf Salmon-0.9.1_linux_x86_64.tar.gz && \
        ln -s $SRC/Salmon-latest_linux_x86_64/bin/salmon $BIN/.





RUN Rscript -e 'source("http://bioconductor.org/biocLite.R");library(BiocInstaller); biocLite("DESeq2", dep = TRUE)'
RUN Rscript -e 'source("http://bioconductor.org/biocLite.R");library(BiocInstaller); biocLite("ape", dep = TRUE)'
RUN Rscript -e 'source("http://bioconductor.org/biocLite.R");library(BiocInstaller); biocLite("ctc", dep = TRUE)'
RUN Rscript -e 'source("http://bioconductor.org/biocLite.R");library(BiocInstaller); biocLite("gplots", dep = TRUE)'
RUN Rscript -e 'source("http://bioconductor.org/biocLite.R");library(BiocInstaller); biocLite("Biobase", dep = TRUE)'
RUN Rscript -e 'source("http://bioconductor.org/biocLite.R");library(BiocInstaller); biocLite("qvalue", dep = TRUE)'
RUN Rscript -e 'source("http://bioconductor.org/biocLite.R");library(BiocInstaller); biocLite("fastclust", dep = TRUE)'
RUN Rscript -e 'source("http://bioconductor.org/biocLite.R");library(BiocInstaller); biocLite("goseq", dep = TRUE)'

## bowtie
WORKDIR $SRC
RUN wget https://sourceforge.net/projects/bowtie-bio/files/bowtie/1.1.2/bowtie-1.1.2-linux-x86_64.zip/download -O bowtie-1.1.2-linux-x86_64.zip && \
	unzip bowtie-1.1.2-linux-x86_64.zip && \
	cp bowtie-1.1.2/bowtie* $BIN


## Trinotate
WORKDIR $SRC
RUN git clone --recursive https://github.com/Trinotate/Trinotate.git


# blast
WORKDIR $SRC
RUN wget ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.5.0/ncbi-blast-2.5.0+-x64-linux.tar.gz && \
    tar xvf ncbi-blast-2.5.0+-x64-linux.tar.gz && \
    cp ncbi-blast-2.5.0+/bin/* /usr/local/bin/

# hmmer3:
WORKDIR $SRC
RUN wget http://eddylab.org/software/hmmer3/3.1b2/hmmer-3.1b2-linux-intel-x86_64.tar.gz && \
    tar xvf hmmer-3.1b2-linux-intel-x86_64.tar.gz && \
    cp hmmer-3.1b2-linux-intel-x86_64/binaries/* /usr/local/bin/


# hmmer2
WORKDIR $SRC
RUN wget http://eddylab.org/software/hmmer/2.3/hmmer-2.3.tar.gz && \
    tar xvf hmmer-2.3.tar.gz && \
    cd hmmer-2.3/ && \
    ./configure --prefix=`pwd` && \
    make && \
    mkdir bin man man/man1 && \
    make install && \
    cp bin/hmmsearch /usr/local/bin/hmmsearch2


## TransDecoder

WORKDIR $SRC
RUN git clone https://github.com/TransDecoder/TransDecoder.git && \
    cd TransDecoder && \
    make


## cleanup in SRC

RUN rm -f $SRC/*.tar.gz $SRC/zip
RUN rm -rf ${TRINITY_HOME}/sample_data

## FASTQC
WORKDIR $SRC
RUN wget http://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.5.zip && \
    unzip fastqc_v0.11.5.zip && \
    chmod 755 /usr/local/src/FastQC/fastqc && \
    ln -s /usr/local/src/FastQC/fastqc /usr/local/bin/.


## Bowtie2
WORKDIR $SRC
RUN wget https://sourceforge.net/projects/bowtie-bio/files/bowtie2/2.2.9/bowtie2-2.2.9-linux-x86_64.zip/download -O bowtie2-2.2.9-linux-x86_64.zip && \
    unzip bowtie2-2.2.9-linux-x86_64.zip && \
    mv bowtie2-2.2.9/bowtie2* /usr/local/bin/ && \
    rm *.zip && \
    rm -r bowtie2-2.2.9


RUN apt-get update && apt-get install -y sqlite3 && apt-get clean

##########################################################################################
## some hacky configuration to allow TrinotateWeb to use the existing apache2 installation

COPY 000-default.conf /etc/apache2/sites-available/

RUN cp -r /usr/local/src/Trinotate/TrinotateWeb/cgi-bin/* /usr/lib/cgi-bin/. && \
        cp -r /usr/local/src/Trinotate/PerlLib/* /usr/lib/cgi-bin/PerlLib/. && \
        mkdir -p /usr/lib/cgi-bin/tmp/tcache && \
	chown -R www-data /usr/lib/cgi-bin/tmp && \
	ln -s /etc/apache2/mods-available/cgi.load /etc/apache2/mods-enabled/cgi.load


# for TrinotateWeb so uses local JS
RUN echo "SetEnv LOCAL_JS 1" >> /etc/apache2/apache2.conf


# tmhmm
WORKDIR $SRC



ADD signalp-4.1.zip $SRC
RUN unzip signalp-4.1.zip && \
    ln -s $SRC/signalp-4.1/signalp /usr/local/bin/signalp && \
    mkdir -p  /usr/local/lib/site_perl && \
    cp /usr/local/src/signalp-4.1/lib/FASTA.pm /usr/local/lib/site_perl/ && \
    cp -r /usr/local/src/signalp-4.1 $BIN/

ADD tmhmm-2.0c.zip $SRC
RUN unzip tmhmm-2.0c.zip && \
    cp -r $SRC/tmhmm-2.0c/bin/* /usr/local/bin/ && \
    cp -r $SRC/tmhmm-2.0c/lib/* /usr/local/lib/



# install SRA toolkit
WORKDIR $SRC

RUN wget https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/2.8.2-1/sratoolkit.2.8.2-1-ubuntu64.tar.gz && \
    tar zxvf sratoolkit.2.8.2-1-ubuntu64.tar.gz && \
    cp -r sratoolkit.2.8.2-1-ubuntu64/bin/* /usr/local/bin/.


ENV TERM=xterm-256color

RUN apt-get update && apt-get install -y dialog screen && apt-get clean

RUN pip install numpy

RUN echo export TRINITY_HOME=${TRINITY_HOME} >> /etc/environment

RUN export LD_LIBRARY_PATH=/usr/local/lib:${LD_LIBRARY_PATH} >> /etc/environment

ENV LD_LIBRARY_PATH /usr/local/lib:${LD_LIBRARY_PATH}


## RSEM

WORKDIR $SRC
RUN wget https://github.com/deweylab/RSEM/archive/v1.3.0.tar.gz && \
    tar xvf v1.3.0.tar.gz && \
        cd RSEM-1.3.0 && \
            make && \
                cp rsem-* $BIN && \
                cp convert-sam-for-rsem $BIN && \
                    cp rsem_perl_utils.pm /usr/local/lib/site_perl/ && \
                        cd ../ && rm -r RSEM-1.3.0
                        


RUN Rscript -e 'source("http://bioconductor.org/biocLite.R");library(BiocInstaller); biocLite("fastcluster", dep = TRUE)'
  
WORKDIR $HOME

RUN echo export LD_LIBRARY_PATH=/usr/local/lib:${LD_LIBRARY_PATH} >> /etc/environment

RUN pip install git+https://github.com/ewels/MultiQC.git

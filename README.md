#! /usr/bin/perl
# kopathway
#get the pathway according to the genname and K number of specie
use utf8;
use strict;
use warnings;
#use Getopt::Long;
die "perl $0 <kegg> <out> <shortname>\n" unless(@ARGV eq 3);
my $sname = $ARGV[2];
open KE,(($ARGV[0] =~ /.*\.gz/) ? "gzip -dc $ARGV[0] |" : $ARGV[0]) or die $!;
my ($apathy,$bpathy,$cpathy,$cid) = ("-", "-", "-", "-");
my %path;
while(<KE>){
        chomp;
        if(/^A<b>(.*)<\/b>/){
                $apathy = $1;
        } elsif (/^B\s+<b>(.*)<\/b>/){
                $bpathy = $1;
        } elsif (/^C/){
                next if ($apathy eq "Human Diseases" and $sname ne "hsa");
                if (/PATH\:$sname(\d{5})/){
                        $cid = $1;
                        my @c = split /\s+/,$_;
                        shift @c;shift @c;pop @c;
                        $cpathy = join(" ",@c);
                        $path{$cid}{apathy} = $apathy;
                        $path{$cid}{bpathy} = $bpathy;
                        $path{$cid}{cpathy} = $cpathy;
                        $path{$cid}{num} = 0;
                        @{$path{$cid}{gene}} = ();
                        @{$path{$cid}{ko}} = ();
                } else {
                        $cid = "-";
                        $cpathy = "-";
                }
        } elsif (/^D/){
                next if ($apathy eq "Human Diseases" and $sname ne "hsa");
                next if ($cid eq "-" and $cpathy eq "-");
                my $k;
                if(/(K\d{5})/){
                        $k = $1;
                } else {
                        next;
                }
                my @d = split /;/,$_;
                my $gene = (split /\s+/,$d[0])[1];
#               print "$gene\n";
#               print "$k\n";die;
                $path{$cid}{num}++;
                push @{$path{$cid}{gene}},$gene;
                push @{$path{$cid}{ko}},$k;
        }
}
close KE;
open OUT,">$ARGV[1]" or die $!;
print OUT "KEGG_A_class\tKEGG_B_class\tPathway\tCount\tPathway ID\tGenes\tKOs\n";
foreach my $koid (sort {$path{$b}{num} <=> $path{$a}{num}} keys %path){
        my $gene = join ";", @{$path{$koid}{gene}};
        my $ko = join "+", @{$path{$koid}{ko}};
        print OUT"$path{$koid}{apathy}\t$path{$koid}{bpathy}\t$path{$koid}{cpathy}\t$path{$koid}{num}\tko$koid\t$gene\t$ko\n";
}
close OUT;

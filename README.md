String::Tools
=====

Various tools for handling strings

NAME
    String::Tools - Various tools for handling strings.

SYNOPSIS
     use String::Tools qw(define is_blank shrink stitch stitcher subst trim);

     my $val = define undef; # ''

     say is_blank(undef);    # 1  (true)
     say is_blank('');       # 1  (true)
     say is_blank("\t\n\0"); # 1  (true)
     say is_blank("0");      # '' (false)

     say shrink("  This is  a    test\n");  # 'This is a test'

     ## stitch ##
     say stitch( qw(This is a test) );  # "This is a test"
     say stitch(
        qw(This is a test), "\n",
        qw(of small proportions)
     );  # "This is a test\nof small proportions"

     # Format some other language in a more readable format,
     # yet keep the resulting string small for transport across the network.
     say stitch(qw(
        SELECT *
          FROM table
         WHERE foo = ?
           AND bar = ?
     ));
     # "SELECT * FROM table WHERE foo = ? AND bar = ?"

     ## subst ##
     my $date = 'Today is ${ day } of $month, in the year $year';
     say subst( $date,   day => '15th', month => 'August', year => 2013   );
     # OR
     say subst( $date, { day => '15th', month => 'August', year => 2013 } );
     # OR
     say subst( $date, [ day => '15th', month => 'August', year => 2013 ] );

     my $lookfor = 'The thing you're looking for is $_.';
     say               subst( $lookfor, 'this' );
     say 'No, wait! ', subst( $lookfor, _ => 'that' );

     say trim("  This is  a    test\n");    # 'This is  a    test'

     # Describe what to trim:
     say trim("  This is  a    test\n",
              l => '\s+', r => '\n+');      # 'This is  a    test'

DESCRIPTION
    "String::Tools" is a collection of tools to manipulate strings.

  Variables
    $THREAD
        The default thread to use while stitching a string together.
        Defaults to a single space, ' '. Used in "shrink( $string )" and
        "stitch( @list )".

    $BLANK
        The default regular expression character class to determine if a
        string component is blank. Defaults to "[[:cntrl:][:space:]]". Used
        in "is_blank( $string )", "shrink( $string )", "stitch( @list )",
        and "trim( $string, qr/l/, qr/r/ )".

  Functions
    define( $scalar )
        Returns $scalar if it is defined, or the empty string if it's
        undefined. Useful in avoiding the 'Use of uninitialized value'
        warnings. $scalar defaults to $_ if not specified.

    is_blank( $string )
        Return true if $string is blank. A blank $string is undefined, the
        empty string, or a string that conisists entirely of "$BLANK"
        characters. $string defaults to $_ if not specified.

    shrink( $string )
        Combine multiple consecutive $BLANK characters into one $THREAD
        character throughout $string. $string defaults to $_ if not
        specified.

    stitch( @list )
        Stitch together the elements of list with "$THREAD". If an item in
        @list is blank (as measured by "is_blank( $string )"), then the item
        is stitched without "$THREAD".

        This approach is more intuitive than "join":

         say   join( ' ' => qw( 1 2 3 ... ), "\n", qw( Can anybody hear? ) );
         # "1 2 3 ... \n Can anybody hear?"
         say   join( ' ' => qw( 1 2 3 ... ) );
         say   join( ' ' => qw( Can anybody hear? ) );
         # "1 2 3 ...\nCan anybody hear?"
         #
         say stitch( ' ' => qw( 1 2 3 ... ), "\n", qw( Can anybody hear? ) );
         # "1 2 3 ...\nCan anybody hear?"

         say   join( ' ' => $user, qw( home dir is /home/ ),     $user );
         # "$user home dir is /home/ $user"
         say   join( ' ' => $user, qw( home dir is /home/ ) ) .  $user;
         # "$user home dir is /home/$user"
         #
         say stitch( ' ' => $user, qw( home dir is /home/ ), '', $user );
         # "$user home dir is /home/$user"

    stitcher( $thread => @list )
        Stitch together the elements of list with $thread in place of
        "$THREAD".

    subst( $string, %variables )
        Take in $string, and do a search and replace of all the variables
        named in %variables with the associated values.

        The %variables parameter can be a hash, hash reference, array
        reference, list, scalar, or empty. The single scalar is treated as
        if the name is the underscore. The empty case is handled by using
        underscore as the name, and $_ as the value.

        If you really want to replace nothing in the string, then pass in an
        empty hash reference or empty array reference, as an empty hash or
        empty list will be treated as the empty case.

        Only names which are in %variables will be replaced. This means that
        substitutions that are in $string which are not mentioned in
        %variables are simply ignored and left as is.

        Returns the string with substitutions made.

    trim( $string, qr/l/, qr/r/ )
        Trim "string" of leading and trailing characters. $string defaults
        to $_ if not specified. The paramters "l" (lead) and "r" (rear) are
        both optional, and can be specified positionally, or as key-value
        pairs. If "l" is undefined, the default pattern is "/$BLANK+/",
        matched at the beginning of the string. If "r" is undefined, the
        default pattern is the value of "l", matched at the end of the
        string.

        If you don't want to trim the start or end of a string, set the
        corresponding parameter to the empty string ''.

         say map trim, @strings;

         say trim('  This is a test  ')
         # 'This is a test'

         say trim('--This is a test==', qr/-/, qr/=/);
         # '-This is a test='

         say trim('  This is a test!!', r => qr/[.?!]+/, l => qr/\s+/);
         # 'This is a test'

BUGS
    None known.

TODO
    Nothing?

AUTHOR
    Bob Kleemann

SEE ALSO
    "join" in perlfunc, Any templating system.

